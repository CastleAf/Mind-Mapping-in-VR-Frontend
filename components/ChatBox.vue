<template>
  <b-form inline class='chat-box' @submit='onSubmit($event)'>
    <b-form-input id="inline-form-input-text" class="mb-2 mr-sm-2 mb-sm-0" placeholder="Insert chat here..."
      v-model='text'></b-form-input>
    <b-button class="mr-2" :disabled='text === ""' type="submit" variant="primary">Send</b-button>
    <b-button @click="$emit('reset')" variant="secondary">Go back</b-button>
  </b-form>
</template>

<script>
export default {
  name: 'ChatBox',
  data: () => ({

    // To keep the state of the input content
    text: ''
  }),
  methods: {

    // We will call this when the form is submitted
    onSubmit(event) {

      // This fires an event which we will handle
      // in the parent component
      this.$emit("submit", event, this.text);
      this.text = '';
    }
  }
}
</script>
<style scoped>
.chat-box {
  width: 95%;
  margin-left: auto;
  margin-right: auto;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 1rem;
  margin-bottom: 1rem;
  font-size: 1.2rem;
  line-height: 1.5rem;
}

input {
  width: min(100%, 20rem);
  flex-grow: 1;
}

button:disabled {
  opacity: 0.5;
}
</style>