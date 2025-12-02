<script>
  import * as Field from "$lib/components/ui/field/index.js";
  import { Button } from "$lib/components/ui/button/index.js";
  // import { Input } from "$lib/components/ui/input/index.js";
  // import { Textarea } from "$lib/components/ui/textarea/index.js";
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
          <p>placeholder</p>
          <!-- <Input id="name" type="text" placeholder="Your Name" /> -->
        </Field.Field>
        <Field.Field class="">
          <p>placeholder</p>
          <!-- <Input
            id="email"
            type="email"
            placeholder="Your Email"
            aria-invalid
          /> -->
          <!-- <Field.Error>This is no Email adress.</Field.Error> -->
        </Field.Field>
        <Field.Field class="">
          <!-- <Textarea id="message" placeholder="Message" rows={4} /> -->
          <p>placeholder</p>
        </Field.Field>
        <Field.Field class="">
          <Button type="submit" class="" disabled={false}>Send</Button>
        </Field.Field>
      </Field.Group>
    </Field.Set>
  </div>
  <div class="contactInfo">
    <h2
      class="scroll-m-20 border-b pb-2 text-3xl font-semibold tracking-tight transition-colors first:mt-0"
    >
      Contact
    </h2>
  </div>
</div>

<style>
  .contact {
    margin: 0 14% 0 6%;
    height: 100%;
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    align-items: center;
    gap: 5rem;
  }

  .contactForm,
  .contactInfo {
  }
</style>
