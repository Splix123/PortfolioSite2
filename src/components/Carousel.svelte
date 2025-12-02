<script>
  import emblaCarouselSvelte from "embla-carousel-svelte";
  import Herosection from "./Sections/Herosection.svelte";
  import AboutMe from "./Sections/AboutMe.svelte";
  import Career from "./Sections/Career.svelte";
  import Projects from "./Sections/Projects.svelte";
  import Contact from "./Sections/Contact.svelte";
  import ProgressBar from "./ProgressBar.svelte";
  import Themeswitcher from "./Themeswitcher.svelte";

  let emblaApi;
  let options = { loop: false };

  let sections = [Herosection, AboutMe, Career, Projects, Contact];

  let scrollProgress = 0;
  let scaleTween;
  let scaleFactor = 0.1;
  let opacityTween;
  let opacityFactor = 0.6;
  let slideRefs = [];

  function onInit(event) {
    emblaApi = event.detail;
    scaleTween = scaleFactor * emblaApi.scrollSnapList().length;
    opacityTween = opacityFactor * emblaApi.scrollSnapList().length;

    emblaApi.on("reInit", () => {
      updateProgress();
      updateScale();
      updateOpacity();
    });

    emblaApi.on("scroll", () => {
      updateProgress();
      updateScale();
      updateOpacity();
    });

    emblaApi.on("slideFocus", () => {
      updateProgress();
      updateScale();
      updateOpacity();
    });

    updateProgress();
    updateScale();
    updateOpacity();
  }

  function updateProgress() {
    if (!emblaApi) return;
    scrollProgress = Math.max(0, Math.min(1, emblaApi.scrollProgress())) * 100;
  }

  function updateScale() {
    if (!emblaApi) return;

    const scrollSnapList = emblaApi.scrollSnapList();
    const slidesInView = emblaApi.slidesInView ? emblaApi.slidesInView() : [];

    scrollSnapList.forEach((snap, snapIndex) => {
      const slidesInSnap = [snapIndex];

      slidesInSnap.forEach((slideIndex) => {
        if (!slideRefs[slideIndex]) return;

        let diffToTarget = snap - emblaApi.scrollProgress();
        const tweenValue = 1 - Math.abs(diffToTarget * scaleTween);
        const scale = Math.min(Math.max(tweenValue, 0), 1);

        slideRefs[slideIndex].style.transform = `scale(${scale})`;
      });
    });
  }

  function updateOpacity() {
    if (!emblaApi) return;

    const scrollSnapList = emblaApi.scrollSnapList();

    scrollSnapList.forEach((snap, index) => {
      let diff = snap - emblaApi.scrollProgress();
      const tweenValue = 1 - Math.abs(diff * opacityTween);
      const opacity = Math.min(Math.max(tweenValue, 0), 1);

      if (slideRefs[index]) {
        slideRefs[index].style.opacity = opacity;
      }
    });
  }
</script>

<div
  class="carousel"
  use:emblaCarouselSvelte={{ options, plugins: [] }}
  onemblaInit={onInit}
>
  <div class="carouselContainer">
    {#each sections as section, i}
      <div class="carouselSlide" bind:this={slideRefs[i]}>
        <svelte:component this={section} />
      </div>
    {/each}
  </div>
  <div class="carouselFooter">
    <ProgressBar progress={scrollProgress} />
    <Themeswitcher />
  </div>
</div>

<style>
  .carousel {
    overflow: hidden;
    display: flex;
    flex-direction: column;
    height: calc(100vh - 18px);
  }

  .carouselContainer {
    height: 100%;
    display: flex;
  }

  .carouselSlide {
    flex: 0 0 75%;
    min-width: 0;
    border: 1px solid var(--foreground);
    transition:
      transform 0.1s ease,
      opacity 0.15s ease;
  }

  .carouselFooter {
    display: flex;
    margin: 0.8rem 0 0 0;
    justify-content: center;
    align-items: center;
    gap: 2rem;
  }
</style>
