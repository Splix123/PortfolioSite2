<script>
  import useEmblaCarousel from "embla-carousel-svelte";
  import WheelGestures from "embla-carousel-wheel-gestures";
  let options = { axis: "y", draggable: false, loop: false };
  let plugins = [WheelGestures({ forceWheelAxis: "y" })];

  let educationSteps = [
    {
      title: "Abitur Europagymnasium Wörth",
      time: "2013-2021",
      description: "Weiterbildung",
      id: "edu1",
      duration: 1,
    },
    {
      title: "Bachelor Wirtschaftsinformatik",
      time: "2021-2026",
      description: "Studium an der Uni",
      id: "edu2",
      duration: 1,
    },
    // {
    //   title: "Master Information Systems",
    //   time: "2026-0000",
    //   description: "Master",
    //   id: "edu3",
    //   duration: 1,
    // },
  ];

  let workSteps = [
    {
      title: "Praktikum Codewerk",
      time: "Jan 2025 - Jun 2025",
      description: "Netzwerk & Admin",
      id: "work1",
      duration: 1,
    },
    {
      title: "Werkstudent Codewerk",
      time: "Jul 2025 - Dez 2025",
      description: "Strategy & Digital Transformation",
      id: "work2",
      duration: 1,
    },
    {
      title: "Praktikum BMW",
      time: "Jul 2025 - Dez 2025",
      description: "Strategy & Digital Transformation",
      id: "work3",
      duration: 1,
    },
    {
      title: "Werkstudent BMW",
      time: "Jan 2026 - Jun 2026",
      description: "IoT Projekte",
      id: "work4",
      duration: 1,
    },
  ];

  let currentEdu = 0;
  let currentWork = 0;
</script>

<div class="career">
  <div class="columns">
    <div class="column" use:useEmblaCarousel={{ options, plugins }}>
      <div class="steps">
        {#each educationSteps as step, i}
          <div class="step">
            <h2>{step.title}</h2>
            <p>{step.time}</p>
            <p>{step.description}</p>
          </div>
        {/each}
      </div>
    </div>
    <div class="column" use:useEmblaCarousel={{ options, plugins }}>
      <div class="steps">
        {#each workSteps as step, i}
          <div class="step">
            <h2>{step.title}</h2>
            <p>{step.time}</p>
            <p>{step.description}</p>
          </div>
        {/each}
      </div>
    </div>
  </div>
</div>

<style>
  .career {
    height: 100%;
  }

  .columns {
    display: flex;
    height: 100%;
    gap: 2rem;
  }

  .column {
    flex: 1;
    overflow: hidden;
  }

  .steps {
    display: flex;
    flex-direction: column;
    align-items: center;
    height: 100%;
    touch-action: pan-y pinch-zoom;
  }

  .step {
    flex: 0 0 100%;
    min-width: 0;
  }
</style>
