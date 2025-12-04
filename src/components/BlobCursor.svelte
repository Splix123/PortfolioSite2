<script>
  import { onMount, onDestroy } from "svelte";
  import gsap from "gsap";

  export let blobType = "circle";
  export let fillColor = "#5227FF";
  export let trailCount = 3;
  export let sizes = [30, 95, 45];
  export let innerSizes = [30, 45, 35];
  export let innerColor = "rgba(255,255,255,0.8)";
  export let opacities = [0.6, 0.6, 0.6];
  export let shadowColor = "rgba(0,0,0,0.75)";
  export let shadowBlur = 5;
  export let shadowOffsetX = 10;
  export let shadowOffsetY = 10;
  export let filterId = "blob";
  export let filterStdDeviation = 30;
  export let filterColorMatrixValues =
    "1 0 0 0 0 0 1 0 0 0 0 0 1 0 0 0 0 0 35 -10";
  export let useFilter = true;
  export let fastDuration = 0.1;
  export let slowDuration = 0.3;
  export let fastEase = "power3.out";
  export let slowEase = "power1.out";
  export let zIndex = 50000;

  let blobs = [];

  function handleMove(e) {
    const x = e.clientX ?? e.touches?.[0]?.clientX;
    const y = e.clientY ?? e.touches?.[0]?.clientY;

    blobs.forEach((el, i) => {
      if (!el) return;

      const isLead = i === 0;

      gsap.to(el, {
        x,
        y,
        duration: isLead ? fastDuration : slowDuration,
        ease: isLead ? fastEase : slowEase,
      });
    });
  }

  onMount(() => {
    // Hide the native cursor globally
    document.body.style.cursor = "none";

    window.addEventListener("mousemove", handleMove);
    window.addEventListener("touchmove", handleMove);

    return () => {
      document.body.style.cursor = "unset";
      window.removeEventListener("mousemove", handleMove);
      window.removeEventListener("touchmove", handleMove);
    };
  });
</script>

<!-- FULLSCREEN LAYER FOR CURSOR -->
<div
  class="fixed top-0 left-0 w-screen h-screen pointer-events-none"
  style="z-index: {zIndex};"
>
  {#if useFilter}
    <svg class="absolute w-0 h-0">
      <filter id={filterId}>
        <feGaussianBlur
          in="SourceGraphic"
          result="blur"
          stdDeviation={filterStdDeviation}
        />
        <feColorMatrix in="blur" values={filterColorMatrixValues} />
      </filter>
    </svg>
  {/if}

  <div
    class="absolute inset-0 pointer-events-none select-none"
    style={useFilter ? `filter: url(#${filterId})` : ""}
  >
    {#each Array(trailCount) as _, i}
      <div
        bind:this={blobs[i]}
        class="absolute will-change-transform transform -translate-x-1/2 -translate-y-1/2"
        style="
          width: {sizes[i]}px;
          height: {sizes[i]}px;
          border-radius: {blobType === 'circle' ? '50%' : '0'};
          background-color: {fillColor};
          opacity: {opacities[i]};
          box-shadow: {shadowOffsetX}px {shadowOffsetY}px {shadowBlur}px 0 {shadowColor};
        "
      >
        <div
          class="absolute"
          style="
            width: {innerSizes[i]}px;
            height: {innerSizes[i]}px;
            top: {(sizes[i] - innerSizes[i]) / 2}px;
            left: {(sizes[i] - innerSizes[i]) / 2}px;
            background-color: {innerColor};
            border-radius: {blobType === 'circle' ? '50%' : '0'};
          "
        ></div>
      </div>
    {/each}
  </div>
</div>
