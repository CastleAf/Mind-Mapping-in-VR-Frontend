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
              <b-form-input
                id="input-1"
                v-model="tempFileName"
                type="text"
                :state="fileName.length > 0"
                placeholder="Enter file name"
                required
              ></b-form-input>
            </b-form-group>
            <b-form-group class="mt-2">
              <h5 class="ml-1" style="font-weight: bold; color: #45555f">
                Please insert your textual information below:
              </h5>
              <b-row class="mt-2">
                <b-col>
                  <b-form-textarea
                    id="textarea-large"
                    v-model="prompt"
                    size="lg"
                    rows="11"
                    placeholder="Input text here..."
                  ></b-form-textarea>
                  <p v-if="prompt.length > 15000" style="color: #dc3545">
                    Prompt too long, please keep it lower than 2000 words.
                    {{ prompt.length }}
                  </p>
                </b-col>
              </b-row>
            </b-form-group>
            <b-form-group>
              <div class="submit mt-3">
                <b-spinner
                  v-if="loading"
                  class="mr-2"
                  variant="primary"
                  label="Spinning"
                ></b-spinner>
                <b-button variant="info" @click="activateChat">Chat</b-button>
              </div>
            </b-form-group>
          </b-col>
        </b-row>
        <b-row class="main-row2" v-else>
          <b-col :cols="colSize" class="app">
            <div class="messages">
              <Message
                v-for="message in messages"
                :key="message.id"
                :class="['message', { right: message.isMine }]"
                :dark="message.isMine"
                :text="message.text"
                :gptLoading="message.gptLoading"
                :author="message.author"
                :mindMap="message.mindMap"
                @scroll="scrollToBottom"
                @showMindMap="showModal"
              />
            </div>
            <br />
            <ChatBox class="chat-box" @submit="onSubmit" @reset="handleReset" />
          </b-col>
        </b-row>
      </b-card>
    </b-container>

    <!-- B modal - Main Modal -->
    <b-modal
      ref="mind-map-modal"
      hide-footer
      no-close-on-backdrop
      title="Generated Mind Map"
      size="huge"
    >
      <p class="my-4" style="font-size: large">
        Below you can observe the mind map that was generated based on the
        prompt. Feel free to add more information by communicating with the GPT
        model.
      </p>
      <div class="text-right my-1">
        <b-button variant="info" @click="switchTable">
          {{ showTable ? 'Switch to Graph View' : 'Switch to Table View' }}
        </b-button>
      </div>
      <div>
        <b-card
          class="d-block text-center w-75 ml-auto mr-auto"
          v-if="showTable"
        >
          <b-table
            striped
            hover
            :per-page="perPage"
            :current-page="currentPage"
            :items="mindMapData"
            small
          >
          </b-table>
          <b-pagination
            v-model="currentPage"
            :total-rows="totalRows"
            :per-page="perPage"
            align="fill"
            size="sm"
            class="my-0"
          ></b-pagination>
        </b-card>
        <b-card v-else class="d-block text-center">
          <div>
            <vue2-org-tree
              labelClassName="bg-tomato"
              :collapsable="false"
              :data="treeData"
            />
          </div>
        </b-card>
        <div>
          <br />
          <hr />
          <br />
          <div>
            <p class="my-4" style="font-size: large">
              Does everything look okay? If so, you can now generate the
              coordinates for each node and export the mind map to Google Drive.
            </p>
          </div>
        </div>
        <div class="mt-3 text-right">
          <b-button variant="primary" @click="hideModal">Close</b-button>
        </div>
      </div>
    </b-modal>

    <b-modal
      ref="error-modal"
      hide-footer
      no-close-on-backdrop
      title="Error"
      size="small"
    >
    <p> There seems to have been a Syntax Error from the GPT.</p>
    <p> Since we're working with an AI model, it is expected that these errors happen sometimes.</p>
    <p> Unfortunately a new chat session will have to be created. Please click the button below to do so.</p>
    <b-button variant="success" @click="newChat"> New Chat </b-button>
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
      showTable: false,
      treeData: {
        label: 'root',
        expand: true,
        children: [
          { label: 'child 1' },
          {
            label: 'child 2',
            children: [{ label: 'child 4' }, { label: 'child 5' }],
          },
          { label: 'child 3' },
        ],
      },
      loading: false,
      mindMapDisplay: false,
      res: '',
      rootNodeError: false,
      items: [],
      colSize: 12,
      tempFileName: '',
      mindMapButton: false,
      fileName: '',
      totalRows: 1,
      currentPage: 1,
      perPage: 10,
      chat: false,
      data: [],
      id: 2,
      messages: [],
      gptHistory: [
        {
          role: 'system',
          content:
            "You are helping the User to build a Mind Map. The user will prompt for you to create a Mind Map based on a text input. When you have the information needed to build the mind map, please answer in the mind map format, such as: [{'NodeId': 'value', 'NodeName': 'value', 'FromNode': 'value', 'NodeLevel': 'value'}, {'NodeId': 'value', 'NodeName': 'value', 'FromNode': 'value', 'NodeLevel': 'value'}].",
        },
      ],
      mindMap: [],
    }
  },
  computed: {
    mindMapData() {
      return this.mindMap.length > 0 ? this.mindMap : []
    },
  },
  methods: {
    reset() {
      this.loading = false
      this.mindMapDisplay = false
      this.colSize = 12
      this.items = []
    },
    async activateChat() {
      this.chat = true

      this.sendMessage({
        text: 'Hello!',
        role: 'user',
        gptLoading: false,
      })

      await this.requestToGpt('Hello!')
    },
    async requestToGpt(message) {
      this.gptHistory.push({ role: 'user', content: message })
      this.sendMessage({
        text: '',
        role: 'assistant',
        gptLoading: true,
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
            gptLoading: false,
          })

          this.checkMindMap(gptAnswer)
        })
        .catch((err) => {
          this.loading = false
          console.log(err)
          
          if (JSON.stringify(err).includes('SyntaxError')) {
            console.log('Taking care of Syntax Error')

            this.showErrorModal()
          }
        })
    },
    // This method will be called when a new message is sent
    onSubmit(event, textv) {
      event.preventDefault()

      this.sendMessage({
        text: textv,
        role: 'user',
        gptLoading: false,
      })

      this.requestToGpt(textv)
    },
    sendMessage(message) {
      const mine = message.role === 'user'
      const auth = message.role
      this.messages.push({
        id: this.id,
        isMine: mine,
        text: message.text,
        author: auth,
        gptLoading: message.gptLoading,
        mindMap: JSON.stringify(message.mindMap),
      })
      this.id++
      this.scrollToBottom()
    },
    handleReset() {
      this.chat = false
    },
    checkMindMap(gptAnswer) {
      this.rootNodeError = false

      if (gptAnswer.includes('NodeId')) {
        // Clean previous Mind Map buttons
        if (this.mindMapButton) {
          this.messages.forEach((element) => {
            element.mindMap = false
          })
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

        // Replace None with "None" FIXME: Check for error here
        gptAnswer = gptAnswer.replace(' None', '"None"')

        // Replace first single quotes (not in words) into double quotes
        gptAnswer = gptAnswer.replace(/(?<!\w)'/g, '"')

        // Replace last single quotes (not in words) into double quotes
        gptAnswer = gptAnswer.replace(/'(?!\w)/g, '"')

        console.log('Before Parse!!')
        console.log(gptAnswer)

        this.mindMap = JSON.parse(gptAnswer)
        this.totalRows = this.mindMap.length

        this.mindMapDisplay = true

        if (mockMessage) {
          this.messages[this.messages.length - 1].text =
            'The generated mind map content has been generated based on the information you provided.'
          this.messages[this.messages.length - 1].mindMap = true
        } else {
          const lastMessage = this.messages[this.messages.length - 1].text
          this.messages[this.messages.length - 1].text = lastMessage.replace(
            unprocessedMindMap,
            ''
          )
          this.messages[this.messages.length - 1].mindMap = true
        }

        console.log('ANSWER')
        console.log(gptAnswer)

        this.generateTree(this.mindMap)

        if (!this.rootNodeError) {
          this.mindMapButton = true
        }
        this.scrollToBottom()
      }
    },
    generateCoordinates() {
      const textv =
        'Can you please add three columns: "x", "y" and "z" which represent 3d coordinates of each node? Please fill them in order for the mind map be a force based graph.'

      this.sendMessage({
        text: textv,
        role: 'user',
        gptLoading: false,
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
      const container = this.$el.querySelector('.messages')
      if (container && container.scrollTop !== null) {
        container.scrollTop = container.scrollHeight
      }
    },
    showModal() {
      this.$refs['mind-map-modal'].show()
    },
    hideModal() {
      this.$refs['mind-map-modal'].hide()
    },
    showErrorModal() {
      this.$refs['error-modal'].show()
    },
    hideErrorModal() {
      this.$refs['error-modal'].hide()
    },
    generateTree(treeData) {
      // Firstly, sort tree by NodeLevel
      treeData.sort((a, b) => (+a.NodeLevel > +b.NodeLevel ? 1 : -1))

      const newData = treeData.map(({ NodeName: label, ...rest }) => ({
        label,
        ...rest,
      }))

      const map = new Map()

      // Create a mapping between NodeId and increasing integers
      for (let i = 0; i < newData.length; i++) {
        map.set(newData[i].NodeId, i)
      }

      // Replace NodeId and FromNode with map values
      for (let i = 0; i < newData.length; i++) {
        newData[i].NodeId = map.get(newData[i].NodeId)
        newData[i].FromNode = map.get(newData[i].FromNode)
      }

      const objsPerLevel = []

      // Then, create array of arrays of objects, per node level
      for (let i = 0; i < treeData.length; i++) {
        if (objsPerLevel[treeData[i].NodeLevel - 1] === undefined) {
          objsPerLevel[treeData[i].NodeLevel - 1] = []
        }
        objsPerLevel[treeData[i].NodeLevel - 1].push(treeData[i])
      }

      console.log('Data to parse')
      console.log(newData)

      this.list_to_tree(newData)
      /* 
      // Finally, create tree
      for (let level = 0; level < objsPerLevel.length; level++) {
        for (let i = 0; i < objsPerLevel[level].length; i++) {
          if (level === '0') {
            finalObj.label = objsPerLevel[level][i].NodeName
            finalObj.id = objsPerLevel[level][i].NodeId
          } else {
            finalObj[level - 1].children.push({
              label: objsPerLevel[level][i].NodeName,
              id: objsPerLevel[level][i].NodeId,
              expand: true,
              children: [],
            })
          }
        }
      } */
    },
    list_to_tree(list) {
      // console.log(list)

      const map = {}
      let node
      const roots = []
      let i

      for (i = 0; i < list.length; i += 1) {
        map[list[i].NodeId] = i // initialize the map
        list[i].children = [] // initialize the children
      }

      console.log(map)
      console.log(list)

      for (i = 0; i < list.length; i += 1) {
        node = list[i]
        if (
          node.FromNode !== '' &&
          node.FromNode !== null &&
          node.FromNode !== undefined &&
          node.FromNode !== 'None'
        ) {
          // if you have dangling branches check that map[node.FromNode] exists
          list[map[node.FromNode]].children.push(node)
        } else {
          roots.push(node)
        }
      }
      console.log('roots time')
      console.log(roots)
      if (roots.length > 1) {
        console.log('There are more than one root node. Printing message...')

        this.rootNodeError = true
        
        const textv = "There seems to be more than one root node. Can you rebuild the mind map, having only one root node?"

        this.sendMessage({
          text: textv,
          role: 'user',
          gptLoading: false,
        })

        this.requestToGpt(textv)
      }
      this.treeData = roots[0]
      return roots
    },
    switchTable() {
      this.showTable = !this.showTable
    },
    async newChat() {

      this.hideErrorModal()
      console.log('Reset on chat')

      // This method is called on the syntax error modal
      // Reset and start a new chat.
      this.messages = []
      this.gptHistory = [
        {
          role: 'system',
          content:
            "You are helping the User to build a Mind Map. The user will prompt for you to create a Mind Map based on a text input. When you have the information needed to build the mind map, please answer in the mind map format, such as: [{'NodeId': 'value', 'NodeName': 'value', 'FromNode': 'value', 'NodeLevel': 'value'}, {'NodeId': 'value', 'NodeName': 'value', 'FromNode': 'value', 'NodeLevel': 'value'}].",
        },
      ]
      this.mindMap = []
      this.id = 2

      this.sendMessage({
        text: 'Hello!',
        role: 'user',
        gptLoading: false,
      })

      await this.requestToGpt('Hello!')

    }
  },
  mounted() {
    /* const treeData = [
      { NodeId: '1', NodeName: 'Embryology', FromNode: '', NodeLevel: '1' },
      { NodeId: '2', NodeName: 'Ontogeny', FromNode: '1', NodeLevel: '2' },
      { NodeId: '3', NodeName: 'Phylogeny', FromNode: '1', NodeLevel: '2' },
      {
        NodeId: '4',
        NodeName: 'Vertebrate Animals',
        FromNode: '2',
        NodeLevel: '3',
      },
      { NodeId: '5', NodeName: 'Fertilization', FromNode: '4', NodeLevel: '4' },
      { NodeId: '6', NodeName: 'Segmentation', FromNode: '5', NodeLevel: '5' },
      {
        NodeId: '7',
        NodeName: 'Differentiation of Cells',
        FromNode: '5',
        NodeLevel: '5',
      },
    ] */
    // this.generateTree(treeData)
    // console.log('here we go')
    // const temp = "[{NodeId: 1, NodeName: Ovaries, FromNode: 0, NodeLevel: 1}, {NodeId: 2, NodeName: Development of Ova, FromNode: 0, NodeLevel: 1}, {NodeId: 3, NodeName: Size and Location of Ova, FromNode: 0, NodeLevel: 1}, {NodeId: 4, NodeName: Structure of Ovum, FromNode: 2, NodeLevel: 2}, {NodeId: 5, NodeName: Coverings of the Ovum, FromNode: 2, NodeLevel: 2}, {NodeId: 6, NodeName: Discharge of Ova, FromNode: 2, NodeLevel: 2}, {NodeId: 7, NodeName: Maturation of the Ovum, FromNode: 2, NodeLevel: 2}, {NodeId: 8, NodeName: Germinal Vesicle, FromNode: 4, NodeLevel: 3}, {NodeId: 9, NodeName: Yolk, FromNode: 4, NodeLevel: 3}]"
    // const abc = temp.replace(/: /g, ': "')
    // const abcTempObj = abc.replace(/,/g, '",')
    // const abcTempObj2 = abcTempObj.replace(/}"/g, '}')
    // const abcObj = abcTempObj2.replace(/}/g, '"}')
    // const abcObj2 = abcObj.replace(/{/g, '{"')
    // const abcObj3 = abcObj2.replace(/:/g, '":')
    // const abcObj4 = abcObj3.replace(/", /g, '", "')
    // console.log(abcObj4)
    // const other = "[{'NodeId': '1', 'NodeName': 'Pleasure and Pain', 'FromNode': '', 'NodeLevel': '0'}, {'NodeId': '2', 'NodeName': 'Introduction and explanation of the mistaken idea that didn't make it', 'FromNode': '1', 'NodeLevel': '1'}, {'NodeId': '3', 'NodeName': 'Account of the system and teachings', 'FromNode': '1', 'NodeLevel': '1'}]"
    // const temp = other.replace(/(?<!\w)'/g, '"')
    // const final = temp.replace(/'(?!\w)/g, '"')
    // console.log(final)
    // console.log(JSON.parse(final))
    // console.log(JSON.parse(abcObj4))
  },
}
</script>

<style>
.modal .modal-huge {
  max-width: 90vw;
  width: 90vw;
  overflow: auto;
}

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
.bg-tomato {
  background-color: #0f3559 !important;
  color: white;
  font-weight: bold;
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

.message + .message {
  margin-top: 1rem;
}

.message.right {
  margin-left: auto;
}
</style>
