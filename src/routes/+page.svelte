<script>
  import Eager from "$lib/Eager.svelte";

  /** @type {HTMLElement | undefined} */
  let bannerEl = $state(undefined);
  let bannerVisible = $state(true);
</script>

{#if bannerVisible}
  <div class="banner" bind:this={bannerEl}>
    <span>This is an important announcement. <a href="/">Learn more</a>.</span>
    <button class="banner-close" onclick={() => (bannerVisible = false)} aria-label="Close banner">
      &times;
    </button>
  </div>
{/if}

<Eager peekOffset={0} {bannerEl}>
  <nav>
    <a href="/">svelte-eager</a>
    <ul>
      <li><a href="#section-1">Section 1</a></li>
      <li><a href="#section-2">Section 2</a></li>
      <li><a href="#section-3">Section 3</a></li>
    </ul>
  </nav>
</Eager>

<div class="sub-bar">
  <button class="active">All</button>
  <button>Drafts</button>
  <button>Published</button>
  <button>Archived</button>
</div>

<main>
  {#each [1, 2, 3] as n}
    <section id="section-{n}">
      <h2>Section {n}</h2>
      <p>
        Lorem ipsum dolor sit amet, consectetur adipiscing elit. Sed do eiusmod tempor incididunt ut
        labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco
        laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in
        voluptate velit esse cillum dolore eu fugiat nulla pariatur.
      </p>
      <p>
        Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit
        anim id est laborum. Sed ut perspiciatis unde omnis iste natus error sit voluptatem
        accusantium doloremque laudantium, totam rem aperiam, eaque ipsa quae ab illo inventore
        veritatis et quasi architecto beatae vitae dicta sunt explicabo.
      </p>
      <p>
        Nemo enim ipsam voluptatem quia voluptas sit aspernatur aut odit aut fugit, sed quia
        consequuntur magni dolores eos qui ratione voluptatem sequi nesciunt. Neque porro quisquam
        est, qui dolorem ipsum quia dolor sit amet, consectetur, adipisci velit.
      </p>
    </section>
  {/each}
</main>

<style>
  :global(*, *::before, *::after) {
    box-sizing: border-box;
  }

  :global(body) {
    margin: 0;
    font-family: system-ui, sans-serif;
    scroll-behavior: smooth;
  }

  nav {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 0 1.5rem;
    height: 3.5rem;
    background: #1a1a2e;
    color: #fff;
    min-height: 120px;
  }

  nav a {
    color: inherit;
    text-decoration: none;
    font-weight: 600;
  }

  ul {
    display: flex;
    gap: 2rem;
    list-style: none;
    margin: 0;
    padding: 0;
  }

  ul a {
    font-weight: 400;
    opacity: 0.8;
  }

  ul a:hover {
    opacity: 1;
  }

  .sub-bar {
    position: sticky;
    /* Tracks the Eager nav's visible height — decreases as the nav collapses,
       so this bar rises to fill the gap until it sticks at the very top. */
    top: var(--eager-visible-height, 0px);
    z-index: 999; /* just below the Eager nav */
    display: flex;
    gap: 0.25rem;
    padding: 0 1.5rem;
    background: #f5f5f7;
    border-bottom: 1px solid #ddd;
  }

  .sub-bar button {
    padding: 0.6rem 1rem;
    background: none;
    border: none;
    border-bottom: 2px solid transparent;
    cursor: pointer;
    font-size: 0.875rem;
    color: #555;
    margin-bottom: -1px; /* overlap the border-bottom of the container */
  }

  .sub-bar button.active {
    border-bottom-color: #1a1a2e;
    color: #1a1a2e;
    font-weight: 600;
  }

  .sub-bar button:hover:not(.active) {
    color: #1a1a2e;
  }

  main {
    max-width: 48rem;
    margin: 0 auto;
    padding: 2rem 1.5rem 4rem;
  }

  section {
    margin-bottom: 4rem;
  }

  h2 {
    font-size: 1.75rem;
    margin-bottom: 1rem;
  }

  p {
    line-height: 1.7;
    color: #444;
    margin-bottom: 1rem;
  }

  .banner {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    z-index: 1001; /* above the Eager nav */
    display: flex;
    align-items: center;
    justify-content: center;
    gap: 1rem;
    padding: 0.6rem 1.5rem;
    background: #e8f4fd;
    border-bottom: 1px solid #b3d8f5;
    font-size: 0.875rem;
    color: #1a3a5c;
    overflow: hidden;
    transition:
      padding 0.3s ease,
      border-width 0.3s ease;
  }

  .banner a {
    color: inherit;
    font-weight: 600;
  }

  .banner-close {
    margin-left: auto;
    background: none;
    border: none;
    font-size: 1.25rem;
    line-height: 1;
    cursor: pointer;
    color: inherit;
    padding: 0 0.25rem;
  }
</style>
