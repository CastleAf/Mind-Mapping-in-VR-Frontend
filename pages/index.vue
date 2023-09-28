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
          <b-row style="">
            <b-col cols="12" xl="6" class="main-description" fluid style="overflow: auto; max-height: fit-content;">
              <h2 style="font-weight: bold">Mind Mapping in VR</h2>
              <br />
              <p>
                This Webapp, developed by
                <a href="mailto:afonso.castelao@tecnico.ulisboa.pt"
                  >Afonso Muller e Sousa Castelão</a
                >
                represents the project to be delivered as his master's thesis
                on Computer Science and Engineering.
              </p>
              <p>
                The app utilizes the
                <a
                  href="https://platform.openai.com/docs/guides/gpt"
                  target="_blank"
                  >OpenAI's GPT</a
                >, a cutting-edge AI language model, to process text and
                generate a structured mind map. It analyzes the content,
                identifies important keywords and concepts, and arranges them
                hierarchically to create a clear mind map.
              </p>
              <p>
                Through the exchange of messages (prompts) with the GPT model, a
                Mind Map will be built. The user is able to keep chatting with the model
                in an informal way in order to achieve the desired result.
              </p>
              <p>
                The final mind map visualization and interaction is to be made
                in a Virtual Reality environment - through an external app
                called <a
                  href="https://noda.io/"
                  target="_blank"
                  >Noda.io</a
                >. When in that environment, the user is able to
                access this app’s generated Mind Map by importing it from google
                drive.
              </p>
              <div class="chat-button">
                To start chatting press the button: 
                <b-button variant="info" @click="activateChat">Start Chat</b-button>
                {{ windowHeight }}
              </div>
            </b-col>
            <b-col cols="12" xl="6">
              <div class="img-div">
                <b-img
                  class="mind-map-img"
                  src="https://media.istockphoto.com/id/1497860763/pt/vetorial/molecular-structure-icon.jpg?s=2048x2048&w=is&k=20&c=BbKEc9x9TMHYxWw5NOPk9igJg6Dpb2NzfU0Y9swB4i8="
                  fluid
                  alt="Responsive image"
                />
              </div>
            </b-col>
          </b-row>
        </div>
        <div class="chat-card" v-else>
          <div style="display: flex; margin-bottom: -10px;"> 
            <h4 class="dark-blue-header ml-2" style="max-width: 67%;">GPT Chat Session:</h4>
            <b-button @click="newChat" variant="secundary" style="padding: 0px 4px; margin-left: auto; margin-right: 0; border: 1px solid rgba(0, 0, 0, 0.25)"><font-awesome-icon :icon="['fas', 'circle-plus']" /> New Chat</b-button>
        </div>
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
          <font-awesome-icon :icon="['fas', 'repeat']" />
          {{ showTable ? 'Toggle Graph View' : 'Toggle Table View' }}
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
        <div class="mb-2">
          <hr />
          <div class="mt-1">
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
        <br />
        <div class="mt-3 d-flex">
          <div class="ml-0 mr-auto">
            <b-button variant="danger" @click="newChat"
              ><font-awesome-icon :icon="['fas', 'circle-plus']" /> New Chat
            </b-button>
          </div>
          <div class="ml-auto mr-0" style="justify-content: flex-end">
            <b-button variant="success" @click="generateCoordinates"
              ><font-awesome-icon :icon="['fas', 'diagram-project']" /> Generate
              Coordinates
            </b-button>
            <b-button variant="primary" @click="hideModal"
              ><font-awesome-icon :icon="['fas', 'comments']" /> Keep
              Chatting</b-button
            >
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
          <b-button variant="danger" @click="newChat"
            ><font-awesome-icon :icon="['fas', 'circle-plus']" /> New Chat
          </b-button>
        </div>
        <div class="ml-auto mr-0">
          <b-button variant="primary" @click="retryChat"
            ><font-awesome-icon :icon="['fas', 'arrow-rotate-left']" /> Retry
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
        It seems the GPT model answered an invalid mind map containing more than
        one root node. A new message asking for the model to rebuild the mind
        map will be sent.
      </p>
      <div class="d-flex">
        <div class="ml-auto mr-0">
          <b-button variant="success" @click="fixMultipleNode"
            ><font-awesome-icon :icon="['fas', 'square-check']" /> OK
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
      windowHeight: '',
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
      this.hideModal()

      // Next time user opens mind map modal, it will be in table view
      this.showTable = true

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
      this.hideMultipleErrorNodeModal()

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
      const textv =
        'The last response was invalid. Please take your time to compute a correctly formatted response (following the JSON format).'

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

    this.windowHeight = window.innerHeight



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
@import url('https://fonts.googleapis.com/css2?family=Nunito+Sans:opsz,wght@6..12,300&display=swap');

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
  font-family: 'Nunito Sans', sans-serif;
  background-image: url('@/assets/img/background.jpg');
  background-repeat: no-repeat;
  background-attachment: fixed;
  background-size: cover;
}

.app-navbar {
  height: fit-content;
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
  padding: 1% 2%;
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
.img-div {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}
.mind-map-img {
  margin-top: auto;
  margin-bottom: auto;
  max-width: 80%;
  max-height: 100%;
}

@media (min-width: 1296px) and (max-width: 1500px) {
  .main-description {
    font-size: 18px;
  }
}

@media (min-width: 1200px) and (max-width: 1295px) {
  .main-description {
    font-size: 17px;
  }
}

@media (max-width: 1199px) {
  .mind-map-img {
    max-width: 85vw;
    height: fit-content;
    width: fit-content;
  }
  .main-description {
    height: fit-content;
    padding: 1% 4%;
  }
  .main-card {
    margin-top: 5vh;
    margin-bottom: 5vh;
    padding: 0%;
    width: 90vw;
    height: fit-content;
  }
  .app-navbar {
    height: fit-content;
  }
  h2 {
    text-align: center;
  }
  .app-container {
    height: 90vh;
  }
  .chat-button {
    text-align: center;
  }
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
