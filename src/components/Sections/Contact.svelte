<script>
  import * as Field from "$lib/components/ui/field/index.js";
  import { Button } from "$lib/components/ui/button/index.js";
  import { Input } from "$lib/components/ui/input/index.js";
  import { Textarea } from "$lib/components/ui/textarea/index.js";

  let name = "";

  let message = "";

  let email = "";
  let emailTouched = false;

  function isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }
</script>

<h1
  class="scroll-m-20 text-4xl font-extrabold tracking-tight lg:text-5xl text-center mt-20"
>
  Contact
</h1>
<div class="contact">
  <div class="contactForm">
    <Field.Set class="">
      <Field.Legend class="">Say Hi!</Field.Legend>
      <Field.Description class="">
        Feel free to reach out for more information or just a friendly hello!
      </Field.Description>
      <Field.Group class="">
        <Field.Field class="">
          <Input
            class=""
            id="name"
            type="text"
            placeholder="Your Name"
            bind:value={name}
          />
        </Field.Field>
        <Field.Field
          class=""
          data-invalid={emailTouched && !isValidEmail(email)}
        >
          <!-- FIXME: invalid not working (on:blur) rest works-->
          <Input
            class=""
            id="email"
            type="email"
            placeholder="Your Email"
            bind:value={email}
            aria-invalid={emailTouched && !isValidEmail(email)}
            on:blur={() => (emailTouched = true)}
          />
          {#if emailTouched && !isValidEmail(email)}
            <Field.Error class="" errors={["This is no Email address."]}>
              This is no Email address.
            </Field.Error>
          {/if}
        </Field.Field>
        <Field.Field class="">
          <Textarea
            class=""
            id="message"
            placeholder="Message"
            rows={4}
            bind:value={message}
          />
        </Field.Field>
        <Field.Field class="">
          <Button type="submit" class="" disabled={false}>Send</Button>
        </Field.Field>
      </Field.Group>
    </Field.Set>
  </div>
  <div class="contactInfo">
    <Field.Set class="">
      <Field.Legend class="">Adresses</Field.Legend>
      <Field.Group class="">
        <Field.Field class="">
          <Field.Label class="">Theresienstraße 68</Field.Label>
          <Field.Label class="">80333 München</Field.Label>
          <Field.Label class="">Germany</Field.Label>
        </Field.Field>
        <Field.Field class="">
          <Field.Label class="">Email:</Field.Label>
          <Field.Label class="">Moritz.ruehm@gmail.com</Field.Label>
          <Field.Label class="">Germany</Field.Label>
        </Field.Field>
        <Field.Legend class="">Follow Me</Field.Legend>
      </Field.Group>
    </Field.Set>
  </div>
</div>

<style>
  .contact {
    margin: 10% 0 0 6%;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    gap: 10%;
  }

  .contactForm,
  .contactInfo {
    width: 45%;
  }
</style>
