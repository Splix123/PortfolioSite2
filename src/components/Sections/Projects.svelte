<script module lang="ts">
  import type { Component } from "svelte";
  import Code from "@lucide/svelte/icons/code";
  import Gamepad2 from "@lucide/svelte/icons/gamepad-2";
  import Music from "@lucide/svelte/icons/music";
  import Paintbrush from "@lucide/svelte/icons/paintbrush";

  interface Project {
    title: string;
    description: string;
    icon: Component<any>;
    image: string;
    link?: string;
  }

  const projects: Project[] = [
    {
      title: "Portfolio Site",
      description:
        "This website — Svelte 5, Tailwind and custom WebGL backgrounds.",
      icon: Code,
      image:
        "https://images.unsplash.com/photo-1520930528075-4ea5ead759f5?w=900&q=80&auto=format&fit=crop",
      link: "https://github.com/Splix123",
    },
    {
      title: "Game Project",
      description: "Placeholder — describe your project in one or two lines.",
      icon: Gamepad2,
      image:
        "https://images.unsplash.com/photo-1511300636408-a63a89df3482?w=900&q=80&auto=format&fit=crop",
      link: "https://example.com",
    },
    {
      title: "Music",
      description: "Placeholder — describe your project in one or two lines.",
      icon: Music,
      image:
        "https://images.unsplash.com/photo-1514519334989-3d5c8b1a9f91?w=900&q=80&auto=format&fit=crop",
      link: "https://soundcloud.com",
    },
    {
      title: "Design Work",
      description: "Placeholder — describe your project in one or two lines.",
      icon: Paintbrush,
      image:
        "https://images.unsplash.com/photo-1778849097774-0ab2a873a858?w=900&q=80&auto=format&fit=crop",
      link: "https://example.com",
    },
  ];

  const CONFIG = {
    width: 680,
    height: 520,
    cardDistance: 60,
    verticalDistance: 70,
    depth: 90,
    skewAmount: 6,
    visibleCards: 4,
    delay: 5000,
    autoplay: true,
    pauseOnHover: true,
    draggable: true,
    speed: 1,
    bounce: 0.35,
    parallax: 0.5,
    hoverSpread: 0.12,
    dim: 0.5,
    cardRadius: 18,
    intro: true,
  };

  // ─── Card stack engine ───────────────────────────────────────

  type Axis = "x" | "y" | "z" | "r" | "p";
  type Vec = { x: number; y: number };
  type Pose = Record<Axis, number>;

  interface DeckCard extends Pose {
    el: HTMLDivElement;
    shade: HTMLSpanElement | null;
    index: number;
    pos: number;
    from: number;
    hold: number;
    nudge: number;
    phase: "rest" | "exit" | "enter" | "drag";
    dir: Vec;
    tilt: number;
    top: number;
    under: boolean;
    v: Pose;
  }

  interface DragState {
    id: number;
    index: number;
    startX: number;
    startY: number;
    active: boolean;
    samples: { x: number; y: number; t: number }[];
  }

  interface ShiftOptions {
    dir?: Vec;
    tilt?: number;
    velocity?: Vec;
  }

  const PERSPECTIVE = 900;
  const TAU = Math.PI * 2;
  const STEP = 1 / 240;
  const PAD = 32;
  const AXES: Axis[] = ["x", "y", "z", "r", "p"];
  const DOWN: Vec = { x: 0, y: 1 };

  const clamp = (value: number, min: number, max: number) =>
    Math.min(max, Math.max(min, value));

  const count = projects.length;
  const visible = clamp(CONFIG.visibleCards, 1, Math.max(1, count));

  const bounds = (() => {
    const { width, height, cardDistance, verticalDistance, depth } = CONFIG;
    const slope = Math.tan((CONFIG.skewAmount * Math.PI) / 180);
    const spread = 1 + CONFIG.hoverSpread;
    let minX = Infinity;
    let maxX = -Infinity;
    let minY = Infinity;
    let maxY = -Infinity;
    for (let i = 0; i < visible; i++) {
      const scale = PERSPECTIVE / (PERSPECTIVE + i * depth);
      for (const u of [-width / 2, width / 2]) {
        for (const v of [-height / 2, height / 2]) {
          const x = (i * cardDistance * spread + u) * scale;
          const y = (-i * verticalDistance * spread + v + u * slope) * scale;
          minX = Math.min(minX, x);
          maxX = Math.max(maxX, x);
          minY = Math.min(minY, y);
          maxY = Math.max(maxY, y);
        }
      }
    }
    return {
      width: Math.ceil(maxX - minX + PAD * 2),
      height: Math.ceil(maxY - minY + PAD * 2),
      originX: PAD - minX,
      originY: PAD - minY,
    };
  })();

  const stageStyle = `width: ${bounds.width}px; height: ${bounds.height}px; perspective: ${PERSPECTIVE}px; transform: translate(-50%, -50%);`;
  const slotStyle = `top: ${bounds.originY}px; left: ${bounds.originX}px; width: ${CONFIG.width}px; height: ${CONFIG.height}px; margin: ${-CONFIG.height / 2}px 0 0 ${-CONFIG.width / 2}px; border-radius: ${CONFIG.cardRadius}px; box-shadow: 0 30px 60px -26px rgb(0 0 0 / 0.55), 0 10px 24px -12px rgb(0 0 0 / 0.35); backface-visibility: hidden;`;

  function startEngine(
    root: HTMLDivElement,
    stage: HTMLDivElement,
    onCardClick: (index: number, isFront: boolean) => void,
  ) {
    const s = CONFIG;
    const reduce =
      window.matchMedia?.("(prefers-reduced-motion: reduce)").matches ?? false;
    const now = performance.now();

    const slots = Array.from(
      stage.querySelectorAll<HTMLDivElement>(":scope > [data-swap-index]"),
    );
    const deck: DeckCard[] = slots.map((el, index) => ({
      el,
      shade: el.querySelector<HTMLSpanElement>(":scope > [data-swap-shade]"),
      index,
      pos: index,
      from: index,
      hold: 0,
      nudge: 0,
      phase: "rest",
      dir: { ...DOWN },
      tilt: 0,
      top: 0,
      under: false,
      x: 0,
      y: 0,
      z: 0,
      r: 0,
      p: 0,
      v: { x: 0, y: 0, z: 0, r: 0, p: 0 },
    }));

    const state = {
      order: deck.map((card) => card.index),
      raf: 0,
      last: 0,
      timer: 0,
      queue: [] as number[],
      layers: 0,
      scale: 1,
      hovered: false,
      overCard: false,
      inView: true,
      drag: null as DragState | null,
      spread: 0,
      spreadV: 0,
      px: 0,
      py: 0,
      pvx: 0,
      pvy: 0,
      tx: 0,
      ty: 0,
      dragged: false,
    };

    const slot = (position: number): Pose => {
      const q = Math.min(position, visible);
      const spread = 1 + state.spread;
      return {
        x: q * s.cardDistance * spread,
        y: -q * s.verticalDistance * spread,
        z: -q * s.depth,
        r: 0,
        p: q,
      };
    };

    const clearance = (dir: Vec) =>
      Math.abs(dir.x) * s.width + Math.abs(dir.y) * s.height + 12;

    const targetOf = (card: DeckCard, time: number): Pose => {
      const base = slot(0);
      if (card.phase === "exit") {
        const reach = clearance(card.dir) * 1.3;
        return {
          x: base.x + card.dir.x * reach,
          y: base.y + card.dir.y * reach,
          z: base.z,
          r: card.tilt,
          p: 0,
        };
      }
      if (card.phase === "enter") {
        return {
          x: base.x,
          y: base.y + clearance(DOWN) * 1.3,
          z: base.z,
          r: 0,
          p: card.p,
        };
      }
      const position = (time < card.hold ? card.from : card.pos) + card.nudge;
      return slot(Math.max(0, position));
    };

    const layerOf = (card: DeckCard) => {
      if (card.top) return 1000 + card.top;
      if (card.under) return 1;
      return 10 + (count - card.pos) * 2;
    };

    const write = (card: DeckCard) => {
      const { el, shade } = card;
      el.style.transform = `translate3d(${card.x.toFixed(2)}px, ${card.y.toFixed(2)}px, ${card.z.toFixed(2)}px) rotate(${card.r.toFixed(3)}deg) skewY(${s.skewAmount}deg)`;
      el.style.zIndex = String(layerOf(card));
      const fade = clamp(visible - card.p, 0, 1);
      el.style.opacity = fade < 1 ? fade.toFixed(3) : "";
      el.style.visibility = fade <= 0.001 ? "hidden" : "";
      if (shade) {
        const depthT = clamp(card.p / Math.max(1, visible - 1), 0, 1);
        shade.style.opacity = (s.dim * 0.62 * depthT).toFixed(3);
      }
      el.style.cursor =
        card.phase === "drag"
          ? "grabbing"
          : card.pos > 0
            ? "pointer"
            : s.draggable
              ? "grab"
              : "";
      el.toggleAttribute("data-front", card.pos === 0);
    };

    const writeStage = () => {
      const ox = bounds.originX + state.px;
      const oy = bounds.originY + state.py;
      stage.style.perspectiveOrigin = `${ox.toFixed(2)}px ${oy.toFixed(2)}px`;
    };

    const spring = (
      value: number,
      velocity: number,
      target: number,
      omega: number,
      zeta: number,
      dt: number,
    ): [number, number] => {
      const k = omega * omega;
      const d = 2 * zeta * omega;
      const steps = Math.max(1, Math.ceil(dt / STEP));
      const h = dt / steps;
      for (let i = 0; i < steps; i++) {
        velocity += (k * (target - value) - d * velocity) * h;
        value += velocity * h;
      }
      return [value, velocity];
    };

    const frame = (time: number) => {
      state.raf = 0;
      const dt = state.last
        ? Math.min(0.05, (time - state.last) / 1000)
        : 1 / 60;
      state.last = time;
      const restOmega = TAU * 1.7 * s.speed;
      const restZeta = reduce ? 1 : 1 - 0.72 * s.bounce;
      const exitOmega = TAU * 1.9 * s.speed;
      let busy = false;

      const spreadTarget = state.hovered && !reduce ? s.hoverSpread : 0;
      const tx = reduce ? 0 : state.tx;
      const ty = reduce ? 0 : state.ty;
      [state.spread, state.spreadV] = spring(
        state.spread,
        state.spreadV,
        spreadTarget,
        TAU * 1.4,
        0.9,
        dt,
      );
      [state.px, state.pvx] = spring(state.px, state.pvx, tx, TAU * 1.1, 1, dt);
      [state.py, state.pvy] = spring(state.py, state.pvy, ty, TAU * 1.1, 1, dt);
      if (
        Math.abs(state.spread - spreadTarget) > 0.0005 ||
        Math.abs(state.spreadV) > 0.0005 ||
        Math.abs(state.px - tx) > 0.05 ||
        Math.abs(state.py - ty) > 0.05 ||
        Math.abs(state.pvx) + Math.abs(state.pvy) > 0.05
      ) {
        busy = true;
      }
      writeStage();

      for (const card of deck) {
        if (card.phase === "drag") {
          busy = true;
          write(card);
          continue;
        }
        const target = targetOf(card, time);
        const moving = card.phase === "exit" || card.phase === "enter";
        const omega = moving ? exitOmega : restOmega;
        const zeta = moving ? 1 : restZeta;
        for (const axis of AXES) {
          [card[axis], card.v[axis]] = spring(
            card[axis],
            card.v[axis],
            target[axis],
            omega,
            zeta,
            dt,
          );
        }

        const base = slot(0);
        if (card.phase === "exit") {
          const travelled =
            (card.x - base.x) * card.dir.x + (card.y - base.y) * card.dir.y;
          if (travelled >= clearance(card.dir)) {
            card.phase = "rest";
            card.top = 0;
            card.hold = 0;
          }
        } else if (card.phase === "enter") {
          if (card.y - base.y >= clearance(DOWN)) {
            card.phase = "rest";
            card.under = false;
            card.hold = 0;
          }
        }

        if (card.hold > time || card.phase !== "rest") busy = true;
        for (const axis of AXES) {
          const limit = axis === "p" ? 0.001 : 0.02;
          if (
            Math.abs(target[axis] - card[axis]) > limit ||
            Math.abs(card.v[axis]) > limit
          )
            busy = true;
        }
        write(card);
      }

      if (busy) {
        state.raf = requestAnimationFrame(frame);
      } else {
        state.last = 0;
        for (const card of deck) {
          if (card.top && card.phase === "rest") card.top = 0;
        }
      }
    };

    const wake = () => {
      if (!state.raf) state.raf = requestAnimationFrame(frame);
    };

    const paused = () =>
      !s.autoplay ||
      count < 2 ||
      (s.pauseOnHover && state.overCard) ||
      state.drag !== null ||
      !state.inView ||
      document.visibilityState === "hidden";

    const schedule = () => {
      clearTimeout(state.timer);
      state.timer = 0;
      if (paused()) return;
      state.timer = window.setTimeout(
        () => {
          state.timer = 0;
          shift(1);
        },
        Math.max(400, s.delay),
      );
    };

    const settleOrder = (time: number, stagger: number) => {
      state.order.forEach((index, position) => {
        const card = deck[index];
        if (card.pos === position) return;
        card.from = card.pos;
        card.pos = position;
        card.hold = time + position * stagger;
      });
    };

    const shift = (direction: number, options: ShiftOptions = {}) => {
      if (count < 2) return;
      const time = performance.now();
      const stagger = 50 / s.speed;
      if (direction > 0) {
        const front = deck[state.order[0]];
        state.order = [...state.order.slice(1), front.index];
        settleOrder(time, stagger);
        front.phase = "exit";
        front.dir = options.dir ?? DOWN;
        front.tilt = options.tilt ?? 0;
        front.top = ++state.layers;
        front.under = false;
        front.nudge = 0;
        front.hold = 0;
        if (options.velocity) {
          front.v.x = options.velocity.x;
          front.v.y = options.velocity.y;
        }
      } else {
        const back = deck[state.order[count - 1]];
        state.order = [back.index, ...state.order.slice(0, count - 1)];
        settleOrder(time, stagger);
        back.phase = "enter";
        back.top = 0;
        back.under = true;
        back.nudge = 0;
        back.hold = 0;
      }
      deck.forEach((card) => {
        if (card.phase === "rest") card.nudge = 0;
      });
      wake();
      schedule();
    };

    const goTo = (position: number) => {
      state.queue.forEach(clearTimeout);
      state.queue = [];
      const gap = 110 / s.speed;
      for (let i = 0; i < position; i++) {
        state.queue.push(window.setTimeout(() => shift(1), i * gap));
      }
    };

    const fit = () => {
      const w = root.clientWidth;
      const h = root.clientHeight;
      const scale = Math.min(
        1,
        w / bounds.width,
        h > 0 ? h / bounds.height : 1,
      );
      state.scale = scale > 0 ? scale : 1;
      stage.style.transform = `translate(-50%, -50%) scale(${state.scale.toFixed(4)})`;
    };

    const cardFrom = (event: Event) =>
      (event.target as Element | null)?.closest?.("[data-swap-index]") ?? null;

    const onMove = (event: PointerEvent) => {
      if (event.pointerType !== "touch") {
        const box = root.getBoundingClientRect();
        const nx = clamp(
          ((event.clientX - box.left) / Math.max(1, box.width)) * 2 - 1,
          -1,
          1,
        );
        const ny = clamp(
          ((event.clientY - box.top) / Math.max(1, box.height)) * 2 - 1,
          -1,
          1,
        );
        state.tx = nx * s.parallax * 0.35 * bounds.width;
        state.ty = ny * s.parallax * 0.35 * bounds.height;
        state.hovered = true;
        const overCard = cardFrom(event) !== null;
        if (overCard !== state.overCard) {
          state.overCard = overCard;
          schedule();
        }
        wake();
      }

      const drag = state.drag;
      if (!drag || drag.id !== event.pointerId) return;
      const dx = (event.clientX - drag.startX) / state.scale;
      const dy = (event.clientY - drag.startY) / state.scale;
      const card = deck[drag.index];
      if (!drag.active) {
        if (Math.hypot(dx, dy) < 5) return;
        drag.active = true;
        state.dragged = true;
        card.phase = "drag";
        card.v = { x: 0, y: 0, z: 0, r: 0, p: 0 };
        root.setPointerCapture?.(event.pointerId);
        root.style.cursor = "grabbing";
      }
      const base = slot(0);
      card.x = base.x + dx;
      card.y = base.y + dy;
      card.z = base.z;
      card.r = clamp(dx * 0.035, -9, 9);
      card.p = 0;
      const time = performance.now();
      drag.samples.push({ x: card.x, y: card.y, t: time });
      while (drag.samples.length > 2 && time - drag.samples[0].t > 90)
        drag.samples.shift();
      const progress = clamp(
        Math.hypot(dx, dy) / (Math.min(s.width, s.height) * 0.3),
        0,
        1,
      );
      const next = deck[state.order[1]];
      if (next && next.phase === "rest") next.nudge = -0.22 * progress;
      wake();
    };

    const onLeave = (event: PointerEvent) => {
      if (event.pointerType === "touch" || state.drag?.active) return;
      state.hovered = false;
      state.overCard = false;
      state.tx = 0;
      state.ty = 0;
      schedule();
      wake();
    };

    const onDown = (event: PointerEvent) => {
      state.dragged = false;
      if (!s.draggable || event.button > 0 || count < 2) return;
      const target = cardFrom(event);
      if (!target) return;
      const index = Number(target.getAttribute("data-swap-index"));
      if (index !== state.order[0] || deck[index].phase !== "rest") return;
      state.drag = {
        id: event.pointerId,
        index,
        startX: event.clientX,
        startY: event.clientY,
        active: false,
        samples: [],
      };
      schedule();
    };

    const onUp = (event: PointerEvent) => {
      const drag = state.drag;
      if (!drag || drag.id !== event.pointerId) return;
      state.drag = null;
      root.style.cursor = "";
      if (!drag.active) {
        schedule();
        return;
      }
      root.releasePointerCapture?.(event.pointerId);
      const card = deck[drag.index];
      const base = slot(0);
      const first = drag.samples[0];
      const last = drag.samples[drag.samples.length - 1];
      const span =
        first && last ? Math.max(0.016, (last.t - first.t) / 1000) : 1;
      const velocity =
        first && last && last !== first
          ? { x: (last.x - first.x) / span, y: (last.y - first.y) / span }
          : { x: 0, y: 0 };
      const offset = { x: card.x - base.x, y: card.y - base.y };
      const distance = Math.hypot(offset.x, offset.y);
      const fling = Math.hypot(velocity.x, velocity.y);
      const next = deck[state.order[1]];
      if (next) next.nudge = 0;
      card.phase = "rest";
      if (
        event.type !== "pointercancel" &&
        (distance > Math.min(s.width, s.height) * 0.3 || fling > 700)
      ) {
        const aim = {
          x: offset.x + velocity.x * 0.12,
          y: offset.y + velocity.y * 0.12,
        };
        const length = Math.hypot(aim.x, aim.y) || 1;
        shift(1, {
          dir: { x: aim.x / length, y: aim.y / length },
          tilt: card.r,
          velocity,
        });
      } else {
        card.v.x = velocity.x;
        card.v.y = velocity.y;
        wake();
        schedule();
      }
    };

    const onClick = (event: MouseEvent) => {
      if (state.dragged) {
        state.dragged = false;
        return;
      }
      const target = cardFrom(event);
      if (!target) return;
      const index = Number(target.getAttribute("data-swap-index"));
      const position = state.order.indexOf(index);
      onCardClick(index, position === 0);
      if (position > 0) goTo(position);
    };

    const onKeyDown = (event: KeyboardEvent) => {
      if (event.key === "ArrowRight" || event.key === "ArrowDown") {
        event.preventDefault();
        shift(1);
      } else if (event.key === "ArrowLeft" || event.key === "ArrowUp") {
        event.preventDefault();
        shift(-1);
      }
    };

    // Keep card drags from also dragging the page carousel (Embla).
    const isolateDrag = (event: Event) => {
      if (cardFrom(event)) event.stopPropagation();
    };

    const onVisibility = () => schedule();

    if (s.intro && !reduce) {
      deck.forEach((card) => {
        card.from = 0;
        card.hold = now + (card.pos * 70) / s.speed;
      });
      stage.animate?.([{ opacity: 0 }, { opacity: 1 }], {
        duration: 420,
        easing: "ease-out",
      });
    } else {
      deck.forEach((card) => Object.assign(card, slot(card.pos)));
    }
    fit();
    writeStage();
    deck.forEach(write);
    wake();

    const resizeObserver = new ResizeObserver(fit);
    resizeObserver.observe(root);
    const intersectionObserver = new IntersectionObserver((entries) => {
      state.inView = entries[entries.length - 1]?.isIntersecting ?? true;
      schedule();
    });
    intersectionObserver.observe(root);

    const listeners: [string, EventListener, AddEventListenerOptions?][] = [
      ["pointermove", onMove as EventListener],
      ["pointerleave", onLeave as EventListener],
      ["pointerdown", onDown as EventListener],
      ["pointerup", onUp as EventListener],
      ["pointercancel", onUp as EventListener],
      ["click", onClick as EventListener],
      ["keydown", onKeyDown as EventListener],
      ["mousedown", isolateDrag],
      ["touchstart", isolateDrag, { passive: true }],
    ];
    listeners.forEach(([type, fn, opts]) =>
      root.addEventListener(type, fn, opts),
    );
    document.addEventListener("visibilitychange", onVisibility);
    schedule();

    return () => {
      cancelAnimationFrame(state.raf);
      clearTimeout(state.timer);
      state.queue.forEach(clearTimeout);
      resizeObserver.disconnect();
      intersectionObserver.disconnect();
      listeners.forEach(([type, fn]) => root.removeEventListener(type, fn));
      document.removeEventListener("visibilitychange", onVisibility);
    };
  }
</script>

<script lang="ts">
  import ArrowUpRight from "@lucide/svelte/icons/arrow-up-right";

  let rootEl = $state<HTMLDivElement | null>(null);
  let stageEl = $state<HTMLDivElement | null>(null);
  let headingBox = $state<HTMLDivElement | null>(null);
  let headingEl = $state<HTMLHeadingElement | null>(null);
  let headingSize = $state(160);

  function openProject(index: number, isFront: boolean) {
    const link = projects[index]?.link;
    if (isFront && link) window.open(link, "_blank", "noopener,noreferrer");
  }

  $effect(() => {
    if (!rootEl || !stageEl || !count) return;
    return startEngine(rootEl, stageEl, openProject);
  });

  $effect(() => {
    const box = headingBox;
    const el = headingEl;
    if (!box || !el) return;
    const fit = () => {
      const current = parseFloat(getComputedStyle(el).fontSize);
      if (el.offsetWidth > 0)
        headingSize = (box.clientHeight * current) / el.offsetWidth;
    };
    fit();
    const observer = new ResizeObserver(fit);
    observer.observe(box);
    document.fonts?.ready.then(fit);
    return () => observer.disconnect();
  });
</script>

<div class="flex h-full w-full gap-6 overflow-hidden p-6">
  <div
    bind:this={headingBox}
    class="relative h-full flex-none"
    style="width: {headingSize * 0.75}px;"
  >
    <h1
      bind:this={headingEl}
      class="absolute top-1/2 left-1/2 m-0 font-display leading-none whitespace-nowrap text-primary uppercase select-none"
      style="font-size: {headingSize}px; transform: translate(-50%, -50%) rotate(-90deg);"
    >
      Projects
    </h1>
  </div>

  <!-- svelte-ignore a11y_no_noninteractive_tabindex -->
  <div
    bind:this={rootEl}
    class="group/swap relative h-full min-w-0 flex-1 outline-none [-webkit-tap-highlight-color:transparent]"
    tabindex="0"
    role="region"
    aria-roledescription="carousel"
    aria-label="Projects"
  >
    <div
      bind:this={stageEl}
      class="pointer-events-none absolute top-1/2 left-1/2"
      style={stageStyle}
    >
      {#each projects as project, index (index)}
        <div
          class="pointer-events-auto absolute isolate touch-pan-y overflow-hidden bg-card text-card-foreground select-none group-focus-visible/swap:data-front:outline-2 group-focus-visible/swap:data-front:outline-offset-[3px] group-focus-visible/swap:data-front:outline-ring"
          style={slotStyle}
          data-swap-index={index}
        >
          <div
            class="flex h-full flex-col gap-2 rounded-[inherit] p-2 ring-1 ring-border ring-inset"
          >
            <div class="relative min-h-0 flex-1 overflow-hidden rounded-[12px]">
              <img
                src={project.image}
                alt={project.title}
                draggable="false"
                class="pointer-events-none absolute inset-0 h-full w-full object-cover"
              />
            </div>
            <div class="flex items-start gap-3 px-1.5 pb-1">
              <span
                class="grid size-8 flex-none place-items-center rounded-full bg-muted"
              >
                <project.icon size={16} strokeWidth={1.8} />
              </span>
              <div class="min-w-0 flex-1">
                <p
                  class="truncate font-heading text-[15px] leading-tight font-semibold"
                >
                  {project.title}
                </p>
                <p
                  class="mt-0.5 line-clamp-2 text-[13px] leading-snug text-muted-foreground"
                >
                  {project.description}
                </p>
              </div>
              {#if project.link}
                <ArrowUpRight
                  class="flex-none text-primary"
                  size={18}
                  strokeWidth={1.8}
                  aria-hidden="true"
                />
              {/if}
            </div>
          </div>
          <span
            data-swap-shade
            class="pointer-events-none absolute inset-0 z-[3] bg-background opacity-0"
            aria-hidden="true"
          ></span>
        </div>
      {/each}
    </div>
  </div>
</div>
