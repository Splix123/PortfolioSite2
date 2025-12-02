<script>
  let sections = [];

  let view;
  function transformScroll(event) {
    view.scrollBy({
      left: event.deltaY,
      behavior: "smooth",
    });
    event.preventDefault();
  }
</script>

<div bind:this={view} class="view" on:wheel={transformScroll}>
  {#each sections as section}
    <section>
      <svelte:component this={section} />
    </section>
  {/each}
</div>

<style>
  * {
    margin: 0;
    padding: 0;
  }
  .view {
    margin-left: -8px;
    margin-top: -8px;
    margin-bottom: -8px;
    margin-right: -8px;
    height: 100vh;
    display: flex;
    overflow-y: hidden;
    -webkit-overflow-scrolling: touch;
    scroll-snap-type: x mandatory;
    scrollbar-width: none;
    -ms-overflow-style: none;
  }

  .view::-webkit-scrollbar {
    display: none;
  }

  section {
    margin-right: -3px;
    border: 1px solid white;
    text-align: center;
    flex: 0 0 100vw;
    scroll-snap-align: start;
  }
</style>
