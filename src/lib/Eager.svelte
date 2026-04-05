<script lang="ts">
  /**
   * Eager navigation bar — behaves as a normal element when scrolling down
   * (slides out of view naturally) and as a sticky element when scrolling up
   * (re-enters the viewport in lock-step with the scroll movement).
   *
   * CSS assumptions on ancestors
   * ─────────────────────────────
   * • The direct parent should establish a normal block formatting context so
   *   the spacer element correctly reserves vertical space.
   * • No ancestor should carry `transform`, `filter`, `perspective`, or
   *   `will-change: transform` — those properties create a new containing
   *   block for fixed-positioned elements, which would break the layout.
   */

  import type { Snippet } from "svelte";

  const clamp = (v: number, min: number, max: number) => Math.max(min, Math.min(max, v));

  let {
    children,
    startHidden = false,
    peekOffset = 0,
  }: {
    children: Snippet;
    startHidden?: boolean;
    peekOffset?: number;
  } = $props();

  let nav = $state<HTMLElement | null>(null);
  let spacer = $state<HTMLElement | null>(null);

  $effect(() => {
    if (!nav || !spacer) return;
    const el_nav = nav;
    const el_spacer = spacer;

    // All position values are in pixels. `offset` is the translateY amount:
    //   0         → nav is fully visible at the top of the viewport (pinned)
    //   minOffset → nav is at its most hidden (peekOffset px still visible)
    let navH = el_nav.offsetHeight;
    let minOffset = -(navH - peekOffset);
    let lastScrollY = window.scrollY;

    // When startHidden is true, hide the nav proportionally to the current
    // scroll position on load. Otherwise always start fully visible (offset 0),
    // which is the expected UX when reloading mid-page.
    let offset = startHidden ? Math.max(minOffset, -lastScrollY) : 0;

    el_spacer.style.height = `${navH}px`;

    // Sets the transform and publishes --eager-visible-height (the nav's
    // current visible height: navH → 0) so sibling sticky elements can use
    // `top: var(--eager-visible-height)` to track just below the nav edge.
    function applyOffset() {
      el_nav.style.transform = `translateY(${offset}px)`;
      document.documentElement.style.setProperty("--eager-visible-height", `${navH + offset}px`);
    }

    applyOffset();

    // ─── Scroll handler ──────────────────────────────────────────────────────
    // A single clamp replaces the explicit FSM:
    //   • scrolling down (delta > 0): offset decreases → nav slides up
    //   • scrolling up   (delta < 0): offset increases → nav slides down
    // The clamp keeps offset within [minOffset, 0], which naturally enforces the
    // hidden / entering / pinned state transitions without extra branches.
    //
    // cancelAnimationFrame + rAF deduplicates to one DOM write per frame:
    // if multiple scroll events fire before the next paint (common on 120 Hz
    // displays), each one cancels the pending frame and reschedules — so only
    // the last event in a given frame actually touches the DOM. scrollY is
    // read inside the rAF callback to get the freshest value at paint time.
    let rafId = 0;
    function handleScroll() {
      cancelAnimationFrame(rafId);
      rafId = requestAnimationFrame(() => {
        const currentScrollY = window.scrollY;
        const delta = currentScrollY - lastScrollY;
        lastScrollY = currentScrollY;

        offset = clamp(offset - delta, minOffset, 0);
        applyOffset();
      });
    }

    // ─── Resize handler ──────────────────────────────────────────────────────
    // ResizeObserver tracks changes to the nav's intrinsic height (e.g. when
    // content changes at responsive breakpoints) so the spacer stays accurate.
    function handleResize() {
      navH = el_nav.offsetHeight;
      minOffset = -(navH - peekOffset);
      el_spacer.style.height = `${navH}px`;
      // Re-clamp in case the new height invalidates the current offset.
      offset = clamp(offset, minOffset, 0);
      applyOffset();
    }

    const ro = new ResizeObserver(handleResize);
    ro.observe(el_nav);

    // `passive: true` lets the browser skip checking for preventDefault(),
    // enabling optimised scrolling and removing jank.
    window.addEventListener("scroll", handleScroll, { passive: true });

    return () => {
      cancelAnimationFrame(rafId);
      ro.disconnect();
      window.removeEventListener("scroll", handleScroll);
      document.documentElement.style.removeProperty("--eager-visible-height");
    };
  });
</script>

<!-- Spacer holds the vertical space the fixed nav would otherwise collapse. -->
<div bind:this={spacer} aria-hidden="true"></div>

<div bind:this={nav} class="eager-nav">
  {@render children()}
</div>

<style>
  .eager-nav {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    /* Expose z-index as a CSS custom property for easy consumer overrides. */
    z-index: var(--eager-z-index, 1000);
    /* Hint to the browser to promote this element to its own compositor layer,
       since its transform is updated on every scroll frame. */
    will-change: transform;
  }
</style>
