<script>
  import "./app.css";
  import { ModeWatcher } from "mode-watcher";
  import Carousel from "./components/Carousel.svelte";
  import BlobCursor from "./components/BlobCursor.svelte";
</script>

<BlobCursor />
<ModeWatcher synchronousModeChanges />
<Carousel />

<style>
</style>
