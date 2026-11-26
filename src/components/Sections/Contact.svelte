<script>
  import * as Field from "$lib/components/ui/field/index.js";
  import { Button } from "$lib/components/ui/button/index.js";
  import { Input } from "$lib/components/ui/input/index.js";
  import { Textarea } from "$lib/components/ui/textarea/index.js";
  import Mail from "@lucide/svelte/icons/mail";
  import MapPin from "@lucide/svelte/icons/map-pin";
  import Copy from "@lucide/svelte/icons/copy";
  import Check from "@lucide/svelte/icons/check";
  import X from "@lucide/svelte/icons/x";
  import Send from "@lucide/svelte/icons/send";
  import ArrowUpRight from "@lucide/svelte/icons/arrow-up-right";
  import FileDown from "@lucide/svelte/icons/file-down";
  import linkedinIcon from "../../assets/icons/linkedin.png";
  import githubIcon from "../../assets/icons/github.png";
  import soundcloudIcon from "../../assets/icons/soundcloud.png";

  const mail = "moritz.ruehm@gmail.com";
  const maxLength = 1000;
  const focusRing =
    "focus-visible:border-primary focus-visible:ring-primary/30";

  const socials = [
    {
      label: "LinkedIn",
      href: "https://www.linkedin.com/in/moritzruehm/",
      icon: linkedinIcon,
      description: "Career &\nnetwork",
      color: "#0A66C2",
    },
    {
      label: "GitHub",
      href: "https://github.com/Splix123",
      icon: githubIcon,
      description: "Code &\nprojects",
      color: "var(--foreground)",
    },
    {
      label: "SoundCloud",
      href: "https://soundcloud.com/moglifast",
      icon: soundcloudIcon,
      description: "My music",
      color: "#FF5500",
    },
  ];

  let name = $state("");
  let message = $state("");
  /** @type {"idle" | "copied" | "error"} */
  let copyStatus = $state("idle");
  let looping = $state(false);

  let canSend = $derived(name.trim() !== "" && message.trim() !== "");
  let nearLimit = $derived(message.length >= maxLength * 0.9);

  let mailto = $derived(
    `mailto:${mail}?subject=${encodeURIComponent(`Portfolio inquiry from ${name.trim()}`)}` +
      `&body=${encodeURIComponent(`${message.trim()}\r\n\r\n– ${name.trim()}`)}`,
  );

  /** @type {ReturnType<typeof setTimeout> | undefined} */
  let copyTimer;

  async function copyEmail() {
    try {
      await navigator.clipboard.writeText(mail);
      copyStatus = "copied";
    } catch {
      copyStatus = "error";
    }
    clearTimeout(copyTimer);
    copyTimer = setTimeout(() => (copyStatus = "idle"), 2000);
  }
</script>

<section
  class="contact-slide @container flex h-full flex-col gap-12 overflow-y-auto px-6 pt-16 pb-8"
>
  <header class="text-center">
    <h1 class="scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl">
      Let's <span class="text-primary">Talk</span>
    </h1>
    <p
      class="mx-auto mt-3 min-h-[3lh] max-w-xl text-balance text-muted-foreground"
    >
      Got a project, a job opportunity or just a question? Write me a few lines
      – I'll get back to you as soon as I can.
    </p>
  </header>

  <div class="flex flex-1 items-center">
    <div
      class="mx-auto grid w-full max-w-5xl gap-8 @xl:grid-cols-2 @4xl:grid-cols-5 @4xl:gap-12"
    >
      <div
        class="min-w-0 rounded-2xl border bg-card p-6 shadow-sm @4xl:col-span-3"
      >
        <Field.Set>
          <Field.Legend>Send me a message</Field.Legend>
          <Field.Group>
            <Field.Field>
              <Field.Label for="name"
                >Name<span class="text-destructive" aria-hidden="true">*</span
                ></Field.Label
              >
              <Input
                class={focusRing}
                id="name"
                required
                type="text"
                autocomplete="name"
                placeholder="Jane Doe"
                bind:value={name}
              />
            </Field.Field>
            <Field.Field>
              <Field.Label for="message"
                >Message<span class="text-destructive" aria-hidden="true"
                  >*</span
                ></Field.Label
              >
              <Textarea
                class={["max-h-48 min-h-28 overflow-y-auto", focusRing]}
                id="message"
                placeholder="Hi Moritz, ..."
                required
                maxlength={maxLength}
                aria-describedby="message-hint message-count"
                bind:value={message}
              />
              <div class="flex items-start justify-between gap-4">
                <Field.Description id="message-hint"
                  >Opens your email app.</Field.Description
                >
                <span
                  id="message-count"
                  class={[
                    "shrink-0 text-xs tabular-nums",
                    nearLimit ? "text-destructive" : "text-muted-foreground",
                  ]}
                >
                  {message.length}/{maxLength}
                </span>
              </div>
            </Field.Field>
            <Field.Field>
              <Button
                href={mailto}
                disabled={!canSend}
                class="aria-disabled:pointer-events-none aria-disabled:opacity-50"
                onmouseenter={() => (looping = true)}
                onanimationend={() => (looping = false)}
              >
                <Send class={["send-plane", looping && "looping"]} />
                Send
              </Button>
            </Field.Field>
          </Field.Group>
        </Field.Set>
      </div>

      <div class="flex min-w-0 flex-col gap-6 @xl:pt-6 @4xl:col-span-2">
        <h2 class="mb-3 text-base font-medium">Other ways to reach me</h2>

        <div class="flex flex-1 flex-col gap-7">
          <div class="flex items-center gap-4">
            <div
              class="grid size-10 shrink-0 place-items-center text-primary [&_svg]:size-5"
              aria-hidden="true"
            >
              <MapPin />
            </div>
            <div class="min-w-0">
              <p class="text-sm font-medium">Location</p>
              <p class="text-sm text-muted-foreground">Cologne, Germany</p>
            </div>
          </div>

          <div class="flex items-center gap-4">
            <div
              class="grid size-10 shrink-0 place-items-center text-primary [&_svg]:size-5"
              aria-hidden="true"
            >
              <Mail />
            </div>
            <div class="min-w-0">
              <p class="text-sm font-medium">Email</p>
              <div class="flex min-w-0 items-center gap-1">
                <a
                  href="mailto:{mail}"
                  class="min-w-0 text-sm text-muted-foreground underline-offset-4 [overflow-wrap:anywhere] hover:text-primary hover:underline"
                >
                  {mail}
                </a>
                <Button
                  variant="ghost"
                  size="icon"
                  class="size-7 shrink-0"
                  onclick={copyEmail}
                  aria-label="Copy email address"
                  title={copyStatus === "error" ? "Copy failed" : undefined}
                >
                  {#if copyStatus === "copied"}
                    <Check />
                  {:else if copyStatus === "error"}
                    <X class="text-destructive" />
                  {:else}
                    <Copy />
                  {/if}
                </Button>
                <span class="sr-only" aria-live="polite">
                  {#if copyStatus === "copied"}Email address copied{:else if copyStatus === "error"}Copying
                    failed{/if}
                </span>
              </div>
            </div>
          </div>

          <div class="flex items-center gap-4">
            <div
              class="grid size-10 shrink-0 place-items-center text-primary [&_svg]:size-5"
              aria-hidden="true"
            >
              <FileDown />
            </div>
            <div class="min-w-0">
              <p class="text-sm font-medium">Resume</p>
              <a
                href="/cv.pdf"
                download="CV_Moritz_Ruehm.pdf"
                class="text-sm text-muted-foreground underline-offset-4 hover:text-primary hover:underline"
              >
                Download CV (PDF)
              </a>
            </div>
          </div>

          <div class="mt-auto flex flex-col gap-3">
            <h3 class="text-sm font-medium">Follow Me</h3>
            <div class="@container">
              <div class="grid grid-cols-3 gap-3">
                {#each socials as social}
                  <a
                    href={social.href}
                    target="_blank"
                    rel="noopener noreferrer"
                    class="social-card group"
                    style="--brand: {social.color}"
                  >
                    <ArrowUpRight
                      aria-hidden="true"
                      class="absolute top-3 right-3 size-4 text-muted-foreground transition-transform group-hover:-translate-y-0.5 group-hover:translate-x-0.5 group-hover:text-(--brand)"
                    />
                    <span
                      class="social-icon"
                      style="mask-image: url({social.icon})"
                      aria-hidden="true"
                    ></span>
                    <span class="social-label">{social.label}</span>
                    <span class="social-desc">{social.description}</span>
                    <span class="sr-only">(opens in new tab)</span>
                  </a>
                {/each}
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>

<style>
  .contact-slide :global(.send-plane.looping) {
    transform-origin: -2px -2px;
    animation: plane-loop 0.7s cubic-bezier(0.45, 0, 0.55, 1);
  }

  @keyframes plane-loop {
    from {
      transform: rotate(0deg);
    }
    to {
      transform: rotate(-360deg);
    }
  }

  @media (prefers-reduced-motion: reduce) {
    .contact-slide :global(.send-plane.looping) {
      animation: none;
    }
  }

  .social-card {
    position: relative;
    min-width: 0;
    min-height: 7.25rem;
    display: flex;
    flex-direction: column;
    padding: 0.75rem;
    border: 1px solid var(--border);
    border-radius: 0.75rem;
    background-color: var(--card);
    transition:
      border-color 150ms,
      transform 150ms,
      box-shadow 150ms;
  }

  .social-card:hover {
    border-color: var(--brand);
    transform: translateY(-2px);
    box-shadow: 0 6px 20px -8px var(--brand);
  }

  .social-card:focus-visible {
    outline: 2px solid var(--brand);
    outline-offset: 2px;
  }

  .social-icon {
    width: 1.75rem;
    height: 1.75rem;
    background-color: var(--brand);
    mask-size: contain;
    mask-repeat: no-repeat;
    mask-position: center;
  }

  .social-label {
    margin-top: 0.75rem;
    font-size: 0.875rem;
    font-weight: 500;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  .social-desc {
    font-size: 0.75rem;
    line-height: 1rem;
    color: var(--muted-foreground);
    white-space: pre-line;
  }

  @container (max-width: 21rem) {
    .social-desc {
      display: none;
    }

    .social-card {
      justify-content: center;
      align-items: center;
      padding: 0.5rem;
    }

    .social-label {
      max-width: 100%;
      font-size: 0.75rem;
    }
  }
</style>
