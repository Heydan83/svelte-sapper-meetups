<script>
  import meetups from "../../meetups-store.js";
  import { createEventDispatcher } from "svelte";
  import TextInput from "../UI/TextInput.svelte";
  import Button from "../UI/BUtton.svelte";
  import Modal from "../UI/Modal.svelte";
  import { isEmpty, isValidEmail } from "../../helpers/validation.js";

  export let id = null;

  let title = "";
  // let titleValid = false;
  let subtitle = "";
  // let subtitleValid = false;
  let address = "";
  // let addressValid = false;
  let email = "";
  // let emailValid = false;
  let description = "";
  // let descriptionValid = false;
  let imageURL = "";
  // let imageURLValid = false;
  // let formIsValid = false;

  if (id) {
    const unsubscribe = meetups.subscribe(items => {
      const selectMeetup = items.find(i => i.id === id);

      title = selectMeetup.title;
      subtitle = selectMeetup.subtitle;
      address = selectMeetup.address;
      email = selectMeetup.contactEmail;
      description = selectMeetup.description;
      imageURL = selectMeetup.imageURL;
    });

    unsubscribe();
  }

  const dispatch = createEventDispatcher();

  $: titleValid = !isEmpty(title);
  $: subtitleValid = !isEmpty(subtitle);
  $: addressValid = !isEmpty(address);
  $: emailValid = isValidEmail(email);
  $: descriptionValid = !isEmpty(description);
  $: imageURLValid = !isEmpty(imageURL);
  $: formIsValid =
    titleValid &&
    subtitleValid &&
    addressValid &&
    emailValid &&
    descriptionValid &&
    imageURL;

  function submitForm() {
    const meetupData = {
      // id: Math.random().toString(),
      title: title,
      subtitle: subtitle,
      description: description,
      imageURL: imageURL,
      contactEmail: email,
      address: address
    };

    // meetups.push(newMeetup); DOES NOT WORK!
    // meetups = [newMeetup, ...meetups];

    if (id) {
      fetch(`https://svelte-course-259921.firebaseio.com/meetups/${id}.json`, {
        method: "PATCH",
        body: JSON.stringify(meetupData),
        headers: { "Content-Type": "application/json" }
      })
        .then(res => {
          if (!res.ok) {
            throw new Error("An Error occurred, please try again!");
          }

          meetups.updateMeetup(id, meetupData);
        })
        .catch(err => {
          console.log(err);
        });
    } else {
      fetch("https://svelte-course-259921.firebaseio.com/meetups.json", {
        method: "POST",
        body: JSON.stringify({ ...meetupData, isFavorite: false }),
        headers: { "Content-Type": "application/json" }
      })
        .then(res => {
          if (!res.ok) {
            throw new Error("An erros occurred, please try again!");
          }

          return res.json();
        })
        .then(data => {
          meetups.addMeetup({ ...meetupData, isFavorit: false, id: data.name });
        })
        .catch(err => {
          console.log(err);
        });
    }

    dispatch("save");
  }

  function deleteMeetup() {
    fetch(`https://svelte-course-259921.firebaseio.com/meetups/${id}.json`, {
      method: "DELETE"
    })
      .then(res => {
        if (!res.ok) {
          throw new Error("An Error occurred, please try again!");
        }

        meetups.removeMeetup(id);
      })
      .catch(err => {
        console.log(err);
      });

    dispatch("save");
  }

  function cancel() {
    dispatch("cancel");
  }
</script>

<style>
  form {
    width: 100%;
  }
</style>

<Modal title="Edit Meetup Data" on:cancel>
  <form on:submit|preventDefault={submitForm}>
    <TextInput
      id="title"
      label="Title"
      valid={titleValid}
      validityMessage="Please enter a valid title."
      value={title}
      on:input={event => {
        title = event.target.value;
      }} />

    <TextInput
      id="subtitle"
      label="Subtitle"
      valid={subtitleValid}
      validityMessage="Please enter a valid subtitle."
      value={subtitle}
      on:input={event => {
        subtitle = event.target.value;
      }} />

    <TextInput
      id="address"
      label="Address"
      valid={addressValid}
      validityMessage="Please enter a valid address."
      value={address}
      on:input={event => {
        address = event.target.value;
      }} />

    <TextInput
      id="imageURL"
      label="Image URL"
      valid={imageURLValid}
      validityMessage="Please enter a valid image URL."
      value={imageURL}
      on:input={event => {
        imageURL = event.target.value;
      }} />

    <TextInput
      id="email"
      label="E-Mail"
      valid={emailValid}
      validityMessage="Please enter a valid E-Mail."
      value={email}
      type="email"
      on:input={event => {
        email = event.target.value;
      }} />

    <TextInput
      id="description"
      label="Description"
      valid={descriptionValid}
      validityMessage="Please enter a valid description."
      controlType="textarea"
      bind:value={description}
      }} />
  </form>
  <div slot="footer">
    <Button type="button" mode="outline" on:click={cancel}>Cancel</Button>
    <Button type="button" on:click={submitForm} disabled={!formIsValid}>
      Save
    </Button>
    {#if id}
      <Button type="button" on:click={deleteMeetup}>Delete</Button>
    {/if}
  </div>
</Modal>
