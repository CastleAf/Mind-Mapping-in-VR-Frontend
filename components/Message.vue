<template>
  <div :class='["message", { dark }]'>
    <h4 class="font-weight-bold text-capitalize">{{ author }} <img class="author-icon" :src="getIcon" alt="*"></h4>
    <span class="message-text">{{ text }}</span> <span v-if="gptLoading"><b-spinner small label="Small Spinner" type="grow"
        variant="primary"></b-spinner>&nbsp;<b-spinner small label="Small Spinner" type="grow"
        variant="primary"></b-spinner>&nbsp;<b-spinner small label="Small Spinner" type="grow"
        variant="primary"></b-spinner></span>
  </div>
</template>

<script>
const assistantIcon = require('../static/assistant.svg')
const userIcon = require('../static/user.svg')

export default {
  name: 'MessageComponent',
  props: [
    'text', // Content of the message
    'author', // Author of the message
    'dark', // Background variant of the box
    'gptLoading' // GPT is processing request
  ],
  computed: {
    getIcon() {
      return this.author === 'user' ? userIcon : assistantIcon
    }
  },
  mounted() {
    console.log('mounted')
    this.$emit('scroll')
  }
}
</script>
<style scoped>
.message {
  background: #e7e7e7;
  color: #0f3559;
  border-radius: 10px;
  padding: 1rem;
  width: fit-content;
  max-width: 90%;
}

.message.dark {
  background: #0f3559;
  color: white;
}

h5 {
  margin: 0 0 .5rem 0;
}

.author-icon {
  width: 1.5rem;
  height: 1.5rem;
  margin-left: .5rem;
  vertical-align: text-top;
}

.message-text {
  font-size: 1.2rem;
  line-height: 1.5rem;
}
</style>