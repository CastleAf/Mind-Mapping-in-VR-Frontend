<template>
  <div class="chat-container">
    <b-form class="chat-box" @submit="onSubmit($event)">
      <b-row class="chat-text">
        <b-form-textarea
          id="textarea"
          v-model="text"
          placeholder="Insert chat here..."
          rows="4"
          max-rows="4"
        ></b-form-textarea>
      </b-row>
      <b-row class="mt-2">
        <div class="chat-buttons">
          <b-button class="mr-2" style="color: #fff;" @click="$emit('reset')" variant="primary"><font-awesome-icon :icon="['fas', 'fa-arrow-left']" style="color: #fff;" /></b-button>
        </div>
        <div class="ml-auto mr-0">
          <b-button
            :disabled="text === ''"
            type="submit"
            variant="primary"
            style="color: #fff;"
            >Send Message <font-awesome-icon :icon="['fas', 'paper-plane']" style="color: #fff;" /></b-button
          >

        </div>
      </b-row>
    </b-form>
  </div>
</template>

<script>
export default {
  name: 'ChatBox',
  data: () => ({
    // To keep the state of the input content
    text: '',
  }),
  methods: {
    // We will call this when the form is submitted
    onSubmit(event) {
      // This fires an event which we will handle
      // in the parent component
      this.$emit('submit', event, this.text)
      this.text = ''
    },
  },
}
</script>
<style scoped>
.chat-box {
  width: 98%;
  margin-left: auto;
  margin-right: auto;
  display: block;
  align-items: center;
  justify-content: center;
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;
  font-size: 1.2rem;
  line-height: 1.5rem;
}

.chat-text {
  width: auto;
  flex-grow: 1;
}

input {
  width: min(100%, 20rem);
  flex-grow: 1;
}

button:disabled {
  opacity: 0.5;
}
.chat-buttons {
  display: flex;
  justify-content: flex-start;
  font-weight: bold;
}
</style>
