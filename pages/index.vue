<template>
  <div class>
    <b-navbar class="app-navbar" toggleable="lg" type="dark" variant="white">
      <b-navbar-brand style="font-weight: bold; color: #45555f">
        <NuxtLogo />
        &nbsp; Mind Mapping In VR
      </b-navbar-brand>
    </b-navbar>
    <b-container class="app-container col d-flex justify-content-center" fluid>
      <!-- Content here -->
      <b-card class="main-card">
        <b-row v-if="!chat" class="main-row">
          <b-col class="main-description ml-3" fluid>
            <h2>Mind Mapping in VR</h2>

            <p>
              Convert Text to Mind Maps: Our web app harnesses the power of
              OpenAI to transform textual information into visual mind maps.
              Simply enter your text, and our app will generate a comprehensive
              mind map that organizes and visualizes the key concepts and
              relationships within your content.
            </p>
            <p>
              How does it work? Our app utilizes the OpenAI API, a cutting-edge
              language model, to process your text and generate a structured
              mind map. It analyzes the content, identifies important keywords
              and concepts, and arranges them hierarchically to create a clear
              and visually appealing mind map.
            </p>
            <p>
              To get started, just type or paste your text into our app, and
              with a click of a button, watch as your information is transformed
              into a table describing a Mind Map.
            </p>
            <p>
              Whether you're looking to organize your thoughts, brainstorm
              ideas, or simplify complex information, our text-to-mind-map
              feature provides a powerful tool to enhance your understanding and
              creativity.
            </p>
          </b-col>
          <b-col v-if="!mindMapDisplay" class="main-form">
            <b-form-group>
              <h5 class="mt-2 ml-1" style="font-weight: bold; color: #45555f">
                Please insert the desired file name:
              </h5>
              <b-form-input id="input-1" v-model="tempFileName" type="text" :state="fileName.length > 0"
                placeholder="Enter file name" required></b-form-input>
            </b-form-group>
            <b-form-group class="mt-2">
              <h5 class="ml-1" style="font-weight: bold; color: #45555f">
                Please insert your textual information below:
              </h5>
              <b-row class="mt-2">
                <b-col>
                  <b-form-textarea id="textarea-large" v-model="prompt" size="lg" rows="11"
                    placeholder="Input text here..."></b-form-textarea>
                  <p v-if="prompt.length > 15000" style="color: #dc3545">
                    Prompt too long, please keep it lower than 2000 words.
                    {{ prompt.length }}
                  </p>
                </b-col>
              </b-row>
            </b-form-group>
            <b-form-group>
              <div class="submit mt-3">
                <b-spinner v-if="loading" class="mr-2" variant="primary" label="Spinning"></b-spinner>
                <b-button :disabled="loading || tempFileName.length == 0" class="mr-2" variant="primary"
                  @click="submit">Submit</b-button>
                <b-button variant="info" @click="activateChat">Chat</b-button>
              </div>
            </b-form-group>
          </b-col>
          <b-col v-else class="main-form">
            <h3>Here you can see a preview of the result:</h3>
            <p>
              This table has been transformed into a csv format and sent to
              Google Drive with the name
              <span class="font-weight-bold">{{ fileName }}.csv</span>.<br />Go
              check the Mind Map in Noda.io app for a visual representation.
            </p>
            <div>
              <b-table striped hover :per-page="perPage" :current-page="currentPage" :items="items" small>
              </b-table>
              <b-pagination v-model="currentPage" :total-rows="totalRows" :per-page="perPage" align="fill" size="sm"
                class="my-0"></b-pagination>
              <b-form-group>
                <div class="text-right mt-3">
                  <b-button variant="primary" @click="reset">New Request</b-button>
                </div>
              </b-form-group>
            </div>
          </b-col>
        </b-row>
        <b-row class="main-row2" v-else>
          <b-col :cols="colSize" class="app">
            <div class="messages">
              <Message v-for="message in messages" :key="message.id" :class="['message', { right: message.isMine }]"
                :dark="message.isMine" :text="message.text" :gptLoading="message.gptLoading" :author="message.author" :mindMap="message.mindMap"
                @scroll="scrollToBottom" @showMindMap="showModal" />
            </div>
            <br>
            <ChatBox class="chat-box" @submit="onSubmit" @reset="handleReset" />
          </b-col>
          <b-col v-if="false" class="main-form">
            <h4>Below you can see a preview of the result:</h4>
            <p>
              This table represents the mind map that was generated based on the prompt. Feel free to add more information
              by communicating with the GPT model.
            </p>
            <div>
              <b-table striped hover :per-page="perPage" :current-page="currentPage" :items="mindMapData" small>
              </b-table>
              <b-pagination v-model="currentPage" :total-rows="totalRows" :per-page="perPage" align="fill" size="sm"
                class="my-0"></b-pagination>
              <b-form-group>
                <div class="text-right mt-3">
                  <b-button class="mr-2" variant="info" @click="generateCoordinates">Generate coordinates</b-button>
                  <b-button class="mr-2" variant="success" @click="doGDriveRequest">Send to GDrive</b-button>
                  <b-button variant="primary" @click="reset">New Request</b-button>
                </div>
              </b-form-group>
            </div>
          </b-col>
        </b-row>
      </b-card>
    </b-container>
    <!-- B modal -->
    <b-modal ref="mind-map-modal" hide-footer no-close-on-backdrop title="Generated Mind Map" size="xl">
      <p class="my-4" style="font-size: large;">
        This table represents the mind map that was generated based on the prompt. Feel free to add more information
        by communicating with the GPT model.
      </p>
      <div class="d-block text-center">
        <b-card>
        <b-table striped hover :per-page="perPage" :current-page="currentPage" :items="mindMapData" small>
        </b-table>
        <b-pagination v-model="currentPage" :total-rows="totalRows" :per-page="perPage" align="fill" size="sm"
          class="my-0"></b-pagination>
        </b-card>
        <div class="mt-3 text-right">
          <b-button variant="primary" @click="hideModal">Close</b-button>
        </div>
      </div>
    </b-modal>
  </div>
</template>

<script>
import ChatBox from '@/components/ChatBox.vue'
import Message from '@/components/Message.vue'

export default {
  name: 'MainPage',
  components: {
    ChatBox,
    Message,
  },
  data() {
    return {
      prompt: '',
      loading: false,
      mindMapDisplay: false,
      res: '',
      items: [],
      colSize: 12,
      tempFileName: '',
      mindMapButton: false,
      fileName: '',
      totalRows: 1,
      currentPage: 1,
      perPage: 15,
      chat: false,
      data: [],
      id: 2,
      messages: [],
      gptHistory: [
        {
          role: "system",
          content: "You are helping the User to build a Mind Map. The user will prompt for you to create a Mind Map based on a text input. When you have the information needed to build the mind map, please answer in the mind map format, such as: [{'NodeId': 'value', 'NomeName': 'value', 'FromNode': 'value', 'NodeLevel': 'value'}, {'NodeId': 'value', 'NomeName': 'value', 'FromNode': 'value', 'NodeLevel': 'value'}].",
        }
      ],
      mindMap: []
    }
  },
  computed: {
    mindMapData() {
      return this.mindMap.length > 0 ? this.mindMap : []
    },
  },
  methods: {
    async submit() {
      this.loading = true
      await this.requestToAPI()
    },
    reset() {
      this.loading = false
      this.mindMapDisplay = false
      this.colSize = 12
      this.items = []
    },
    async requestToAPI() {
      this.tempFileName = this.tempFileName.replace(/ /g, '_')

      // Avoid incorrect name displaying in case user changes
      // filename while request is loading
      this.fileName = this.tempFileName

      const url = '/request/' + this.fileName
      console.log(url)
      await this.$axios
        .$post(url, {
          val: this.prompt,
        })
        .then((res) => {
          this.loading = false
          const results = res.result
          console.log('results')
          console.log(results)
          this.items = results
          this.mindMapDisplay = true
          this.totalRows = this.items.length
          this.res = res
        })
        .catch((err) => {
          this.loading = false
          console.log(err)
        })
    },
    activateChat() {
      this.chat = true
    },
    async requestToGpt(message) {

      this.gptHistory.push({ role: 'user', content: message })
      this.sendMessage({
        text: '',
        role: 'assistant',
        gptLoading: true
      })

      // Request to OpenAI
      const url = '/requestV2/' + 'test'
      await this.$axios
        .$post(url, this.gptHistory)
        .then((res) => {
          console.log(res)
          this.messages.pop()
          const gptAnswer = res.answer.text

          this.gptHistory.push({ role: 'assistant', content: gptAnswer })
          this.sendMessage({
            text: gptAnswer,
            role: res.answer.role,
            gptLoading: false
          })

          this.checkMindMap(gptAnswer)

        })
        .catch((err) => {
          this.loading = false
          console.log(err)
        })
    },
    // This method will be called when a new message is sent
    onSubmit(event, textv) {
      event.preventDefault()

      this.sendMessage({
        text: textv,
        role: 'user',
        gptLoading: false
      })

      this.requestToGpt(textv)
    },
    sendMessage(message) {
      const mine = message.role === 'user'
      const auth = message.role
      this.messages.push({ id: this.id, isMine: mine, text: message.text, author: auth, gptLoading: message.gptLoading, mindMap: JSON.stringify(message.mindMap) },)
      this.id++
      this.scrollToBottom()
    },
    handleReset() {
      this.chat = false
    },
    checkMindMap(gptAnswer) {
      if (gptAnswer.includes('NodeId')) {

        // Clean previous Mind Map buttons
        if (this.mindMapButton) {
          this.messages.forEach(element => {
            element.mindMap = false
          });
        }

        // Flag used in case the GPT Answer contains only the mind map
        let mockMessage = false

        // Search for first '[' in gptAnswer
        const index = gptAnswer.indexOf('[')

        // Remove everything before '['
        gptAnswer = gptAnswer.substring(index)

        // Search for first ']' in gptAnswer
        const index2 = gptAnswer.indexOf(']')

        // Original Mind Map answer
        const unprocessedMindMap = gptAnswer.substring(0, index2 + 1)

        if (unprocessedMindMap.length === gptAnswer.length) {
          console.log('Will print generic message on chat.')
          mockMessage = true
        }

        // Remove everything after ']'
        gptAnswer = gptAnswer.substring(0, index2 + 1)

        // Remove all single quotes
        gptAnswer = gptAnswer.replace(/'/g, '"')

        // Replace None with "None"
        gptAnswer = gptAnswer.replace(/None/g, '"None"')

        this.mindMap = JSON.parse(gptAnswer)
        this.totalRows = this.mindMap.length

        this.mindMapDisplay = true
        // this.colSize = 7

        if (mockMessage) {
          this.messages[this.messages.length - 1].text = 'The generated mind map content has been generated based on the information you provided.'
          this.messages[this.messages.length - 1].mindMap = true
        }
        else {
          const lastMessage = this.messages[this.messages.length - 1].text
          this.messages[this.messages.length - 1].text = lastMessage.replace(unprocessedMindMap, "");
          this.messages[this.messages.length - 1].mindMap = true
        }

        this.mindMapButton = true
        this.scrollToBottom()
      }
    },
    generateCoordinates() {
      const textv = 'Can you please add three columns: "x", "y" and "z" which represent 3d coordinates of each node? Please fill them in order for the mind map be a force based graph.'

      this.sendMessage({
        text: textv,
        role: 'user',
        gptLoading: false
      })

      this.requestToGpt(textv)

    },
    async doGDriveRequest() {
      this.loading = true

      const url = '/requestGDrive/' + 'test'
      await this.$axios
        .$post(url, this.mindMap)
        .then((res) => {
          console.log(res)
        })
        .catch((err) => {
          this.loading = false
          console.log(err)
        })
    },
    scrollToBottom() {
      const container = this.$el.querySelector(".messages")
      container.scrollTop = container.scrollHeight;
    },
    showModal() {
      this.$refs['mind-map-modal'].show()
    },
    hideModal() {
      this.$refs['mind-map-modal'].hide()
    },
  }
}
</script>

<style>
body {
  background-image: url('@/assets/img/background.jpg');
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-size: cover;
}

.app-navbar {
  height: 6vh;
}

.main-row {
  margin-top: 5vh;
}

.main-row2 {
  margin-top: 0vh;
  margin-bottom: 0vh;
}

.main-form {
  height: 55vh;
  padding: 2%;
  color: rgb(69, 85, 95);
}

.main-card {
  margin-top: 15vh;
  margin-bottom: 15vh;
  padding: 0%;
  width: 90vw;
}

.main-description {
  padding: 2%;
  height: 55vh;
  color: rgb(69, 85, 95);
  font-size: 20px;
  font-weight: bolder;
  text-align: justify;
}

.submit {
  display: flex;
  justify-content: flex-end;
}
</style>
<style scoped>
.app {
  height: 60vh;
  width: 90vw;
  padding: 2%;
  padding-bottom: 0%;
  display: flex;
  flex-direction: column;
}

.messages {
  flex-grow: 1;
  overflow: auto;
  padding: 1rem;
}

.message+.message {
  margin-top: 1rem;
}

.message.right {
  margin-left: auto;
}
</style>