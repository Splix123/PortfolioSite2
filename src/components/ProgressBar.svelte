<script>
  export let progress = 0;
</script>

<div class="progress">
  <div
    class="progressBar"
    style="transform: translateX({progress - 100}%);"
  ></div>
</div>

<style>
  .progress {
    margin: 1.5rem 0 0.8rem 0;
    position: relative;
    justify-self: center;
    align-self: center;
    height: 0.6rem;
    width: 13rem;
    max-width: 90%;
    background-color: var(--background-site);
    border-radius: 1.8rem;
    box-shadow: inset 0 0 0 0.1rem grey;
    overflow: hidden;
  }

  .progressBar {
    background-color: white;
    position: absolute;
    top: 0;
    bottom: 0;
    left: 0;
    width: 100%;
    transition: transform 0.2s ease;
  }
</style>
