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
        <div v-if="!chat">
          <b-row class="">
            <b-col class="main-description ml-3" fluid>
              <h2>Mind Mapping in VR</h2>

              <p>
                Convert Text to Mind Maps: Our web app harnesses the power of
                OpenAI to transform textual information into visual mind maps.
                Simply enter your text, and our app will generate a
                comprehensive mind map that organizes and visualizes the key
                concepts and relationships within your content.
              </p>
              <p>
                How does it work? Our app utilizes the OpenAI API, a
                cutting-edge language model, to process your text and generate a
                structured mind map. It analyzes the content, identifies
                important keywords and concepts, and arranges them
                hierarchically to create a clear and visually appealing mind
                map.
              </p>
              <p>
                To get started, just type or paste your text into our app, and
                with a click of a button, watch as your information is
                transformed into a table describing a Mind Map.
              </p>
              <p>
                Whether you're looking to organize your thoughts, brainstorm
                ideas, or simplify complex information, our text-to-mind-map
                feature provides a powerful tool to enhance your understanding
                and creativity.
              </p>
              <b-button variant="info" @click="activateChat">Chat</b-button>
            </b-col>
          </b-row>
        </div>
        <div class="chat-card" v-else>
          <h4 class="dark-blue-header ml-2">GPT Chat Session:</h4>
          <hr />
          <b-row class="chat-row">
            <b-col :cols="colSize" class="chat-col">
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
              <ChatBox
                class="chat-box"
                @submit="onSubmit"
                @reset="handleReset"
              />
            </b-col>
          </b-row>
        </div>
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
      <div class="text-right my-2">
        <b-button variant="success" @click="switchTable">
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
          <div class="org-tree-div">
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
            <p class="mx-4 mb-0" style="font-size: large">
              Above you can observe the mind map that was generated based on the
              prompt. Feel free to add more information by communicating with
              the GPT model.
            </p>
            <p class="mx-4" style="font-size: large">
              Does everything look okay? If so, you can now generate the
              coordinates for each node and export the mind map to Google Drive.
            </p>
          </div>
        </div>
        <div class="mt-3 d-flex">
          <div class="ml-0 mr-auto">
            <b-button variant="danger" @click="newChat"
              ><font-awesome-icon :icon="['fas', 'circle-plus']" /> New Chat
            </b-button>
          </div>
          <div class="ml-auto mr-0">
            <b-button variant="primary" @click="hideModal">Close</b-button>
          </div>
      </div>
      </div>
    </b-modal>

    <b-modal
      ref="error-modal"
      centered
      hide-footer
      no-close-on-backdrop
      title="Error"
      size="small"
    >
      <p>There seems to have been some kind of error from the GPT response.</p>
      <p>
        Since we're working with an AI model, it is expected that these errors
        happen sometimes.
      </p>
      <p>
        You can either retry to send the last message or start a new chat. In
        case the error keeps happening, please restart the chat.
      </p>
      <div class="d-flex">
        <div class="ml-0 mr-auto">
          <b-button variant="primary" @click="retryChat"
            ><font-awesome-icon :icon="['fas', 'arrow-rotate-left']" /> Retry
          </b-button>
        </div>
        <div class="ml-auto mr-0">
          <b-button variant="danger" @click="newChat"
            ><font-awesome-icon :icon="['fas', 'circle-plus']" /> New Chat
          </b-button>
        </div>
      </div>
    </b-modal>

    <b-modal
      ref="error-multiple-node-modal"
      centered
      hide-footer
      no-close-on-backdrop
      title="Error"
      size="small"
    >
      <p>
        It seems the GPT model answered an invalid mind map containing more than one root node.
        A new message asking for the model to rebuild the mind map will be sent.
      </p>
      <div class="d-flex">
        <div class="ml-auto mr-0">
          <b-button variant="success" @click="fixMultipleNode"
            ><font-awesome-icon :icon="['fas', 'square-check']" /> Ok
          </b-button>
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

      if (!this.messages.length) {
        this.sendMessage({
          text: 'Hello!',
          role: 'user',
          gptLoading: false,
        })

        await this.requestToGpt('Hello!')
      }
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
          console.log('iommg')

          this.showErrorModal()
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
        gptAnswer = gptAnswer.replace(' None', ' "None"')

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
    showMultipleErrorNodeModal() {
      this.$refs['error-multiple-node-modal'].show()
    },
    hideMultipleErrorNodeModal() {
      this.$refs['error-multiple-node-modal'].hide()
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

        // Fire error-node-modal
        this.showMultipleErrorNodeModal()
      }
      this.treeData = roots[0]
      return roots
    },
    switchTable() {
      this.showTable = !this.showTable
    },
    async fixMultipleNode() {

      const textv =
        'There seems to be more than one root node. Can you rebuild the mind map, having only one root node?'

      this.sendMessage({
        text: textv,
        role: 'user',
        gptLoading: false,
      })

      await this.requestToGpt(textv)
    },
    async retryChat() {
      this.hideErrorModal()
      console.log('Retrying chat')
      const textv = 'The last response was invalid. Please take your time to compute a correctly formatted response.'

      this.sendMessage({
        text: textv,
        role: 'user',
        gptLoading: false,
      })

      await this.requestToGpt(textv)
    },
    async newChat() {
      this.hideModal()
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
    },
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
  max-width: 95vw;
  width: 85vw;
  overflow: auto;
}

.org-tree-container {
  overflow: auto;
}

.chat-div {
  border: 2px solid #0f3559;
  margin-top: 5vh;
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

.main-form {
  height: 60vh;
  padding: 2%;
  color: rgb(69, 85, 95);
}

.main-card {
  margin-top: 12.5vh;
  padding: 0%;
  width: 90vw;
}

.main-description {
  padding: 2%;
  height: 65vh;
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
  background-color: #599cda;
  color: white;
  font-weight: bold;
  border-color: #599cda;
}
.org-tree-div {
  overflow: auto;
  max-height: 40vh;
}
</style>
<style scoped>
.dark-blue-header {
  color: #0f3559;
  font-weight: bold;
}
.chat-col {
  padding: 0 2% 0 2%;
  height: 60vh;
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
