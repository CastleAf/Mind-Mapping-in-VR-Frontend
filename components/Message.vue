<template>
  <div :class='["message", { dark }]'>
    <h5 class="font-weight-bold text-capitalize">{{ author }} <img class="author-icon" :src="getIcon" alt="*"></h5>
    <span class="message-text">{{ text }}</span> <span v-if="gptLoading"><b-spinner small label="Small Spinner" type="grow"
        variant="primary"></b-spinner>&nbsp;<b-spinner small label="Small Spinner" type="grow"
        variant="primary"></b-spinner>&nbsp;<b-spinner small label="Small Spinner" type="grow"
        variant="primary"></b-spinner></span>
    <div v-if="mindMap" class="mt-2">
      <b-button @click="showMindMap" variant="success">Open Mind Map</b-button>
    </div>
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
    'gptLoading', // GPT is processing request
    'mindMap' // Boolean to check if mind map modal button should be shown
  ],
  computed: {
    getIcon() {
      return this.author === 'user' ? userIcon : assistantIcon
    }
  },
  mounted() {
    this.$emit('scroll')
  },
  methods: {
    showMindMap() {
      this.$emit('showMindMap')
    }
  }
}
</script>
<style scoped>
.message {
  font-family: 'Inconsolata', sans-serif;
  font-size: 1rem;
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
  margin: 0 0 .4rem 0;
}

.author-icon {
  width: 1.25rem;
  height: 1.25rem;
  margin-left: .5rem;
  vertical-align: text-bottom;
}

.message-text {
  font-size: 1.1rem;
  line-height: 1.5rem;
}
</style>