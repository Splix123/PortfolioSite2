<script>
  import "./app.css";
  import { ModeWatcher, mode } from "mode-watcher";
  import Carousel from "./components/Carousel.svelte";
  import BlobCursor from "./components/BlobCursor.svelte";
  import BackgroundAnimation from "./components/BackgroundAnimation.svelte";
  import AltBackgroundAnimation from "./components/AltBackgroundAnimation.svelte";

  // Which background to show: "grainient" or "crt"
  const background = "crt";
</script>

<BlobCursor />
<ModeWatcher synchronousModeChanges />

<div class="pointer-events-none fixed inset-0 z-0" aria-hidden="true">
  {#if background === "grainient"}
    <AltBackgroundAnimation
      colors={["#080808", "#1A1A1A", "#454545", "#B51F2E", "#E63846"]}
      lightMode={mode.current === "light"}
      angle={55}
      flow={0.8}
      silk={0.35}
      grainAmount={0.08}
    />
  {:else}
    <BackgroundAnimation
      color="#E63846"
      backgroundColor={mode.current === "light" ? "#F5F3F0" : "#080808"}
      lightMode={mode.current === "light"}
      mask="aperture"
      curvature={0.25}
      persistence={0.4}
      changeOnClick={false}
      dpr={1}
    />
  {/if}
</div>

<div class="relative z-10">
  <Carousel />
</div>

<style>
</style>
