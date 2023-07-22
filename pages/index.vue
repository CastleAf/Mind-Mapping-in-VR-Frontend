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
        <b-row class="main-row">
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
          <b-col v-if="!requested" class="main-form">
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
                <b-button
                  :disabled="loading || tempFileName.length == 0"
                  variant="primary"
                  @click="submit"
                  >Submit</b-button
                >
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
              <b-table
                striped
                hover
                :per-page="perPage"
                :current-page="currentPage"
                :items="items"
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
              <b-form-group>
                <div class="text-right mt-3">
                  <b-button variant="primary" @click="reset"
                    >New Request</b-button
                  >
                </div>
              </b-form-group>
            </div>
          </b-col>
        </b-row>
      </b-card>
    </b-container>
  </div>
</template>

<script>
export default {
  name: 'IndexPage',
  data() {
    return {
      prompt: '',
      loading: false,
      requested: false,
      res: '',
      items: [],
      tempFileName: '',
      fileName: '',
      totalRows: 1,
      currentPage: 1,
      perPage: 9,
    }
  },
  methods: {
    async submit() {
      this.loading = true
      await this.requestToAPI()
    },
    reset() {
      this.loading = false
      this.requested = false
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
          this.requested = true
          this.totalRows = this.items.length
          this.res = res
        })
        .catch((err) => {
          this.loading = false
          console.log('errrr')
          console.log(err)
        })
    },
  },
}
</script>

<style>
body {
  background-image: url('@/assets/img/background.jpg');
}

.app-navbar {
  height: 6vh;
}

.main-row {
  margin-top: 5vh;
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
