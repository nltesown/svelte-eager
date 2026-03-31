<script>
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

  let { children } = $props();

  /** @type {HTMLElement | null} */
  let nav = $state(null);
  /** @type {HTMLElement | null} */
  let spacer = $state(null);

  $effect(() => {
    if (!nav || !spacer) return;

    // All position values are in pixels. `offset` is the translateY amount:
    //   0       → nav is fully visible at the top of the viewport (pinned)
    //   -navH   → nav is completely above the viewport (hidden)
    let navH = nav.offsetHeight;
    let lastScrollY = window.scrollY;

    // Handle the case where the page loads with an existing scroll position.
    let offset = Math.max(-navH, Math.min(0, -lastScrollY));

    spacer.style.height = `${navH}px`;
    nav.style.transform = `translateY(${offset}px)`;

    // ─── Scroll handler ──────────────────────────────────────────────────────
    // A single clamp replaces the explicit FSM:
    //   • scrolling down (delta > 0): offset decreases → nav slides up
    //   • scrolling up   (delta < 0): offset increases → nav slides down
    // The clamp keeps offset within [−navH, 0], which naturally enforces the
    // hidden / entering / pinned state transitions without extra branches.
    function handleScroll() {
      const currentScrollY = window.scrollY;
      const delta = currentScrollY - lastScrollY;
      lastScrollY = currentScrollY;

      offset = Math.max(-navH, Math.min(0, offset - delta));
      nav.style.transform = `translateY(${offset}px)`;
    }

    // ─── Resize handler ──────────────────────────────────────────────────────
    // ResizeObserver tracks changes to the nav's intrinsic height (e.g. when
    // content changes at responsive breakpoints) so the spacer stays accurate.
    function handleResize() {
      navH = nav.offsetHeight;
      spacer.style.height = `${navH}px`;
      // Re-clamp in case the new height invalidates the current offset.
      offset = Math.max(-navH, Math.min(0, offset));
      nav.style.transform = `translateY(${offset}px)`;
    }

    const ro = new ResizeObserver(handleResize);
    ro.observe(nav);

    // `passive: true` lets the browser skip checking for preventDefault(),
    // enabling optimised scrolling and removing jank.
    window.addEventListener('scroll', handleScroll, { passive: true });

    return () => {
      ro.disconnect();
      window.removeEventListener('scroll', handleScroll);
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
