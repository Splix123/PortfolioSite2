<script>
  import SunIcon from "@lucide/svelte/icons/sun";
  import MoonIcon from "@lucide/svelte/icons/moon";

  import { toggleMode, mode } from "mode-watcher";
  import { Button } from "$lib/components/ui/button/index.js";

  function switchTheme() {
    const apply = () => {
      toggleMode();
      mode.current;
    };

    if (
      !document.startViewTransition ||
      window.matchMedia("(prefers-reduced-motion: reduce)").matches
    ) {
      apply();
      return;
    }

    document.startViewTransition(apply);
  }
</script>

<Button onclick={switchTheme} variant="outline" size="icon">
  <SunIcon
    class="h-[1.2rem] w-[1.2rem] rotate-0 scale-100 !transition-all dark:-rotate-90 dark:scale-0"
  />
  <MoonIcon
    class="absolute h-[1.2rem] w-[1.2rem] rotate-90 scale-0 !transition-all dark:rotate-0 dark:scale-100"
  />
  <span class="sr-only">Toggle theme</span>
</Button>

<style>
  :global(::view-transition-old(root)),
  :global(::view-transition-new(root)) {
    animation-duration: 0.5s;
    animation-timing-function: ease-in-out;
  }
</style>
