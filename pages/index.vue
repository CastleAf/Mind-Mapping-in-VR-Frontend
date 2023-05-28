<template>
  <div class>
    <b-navbar class="app-navbar" toggleable="lg" type="dark" variant="white">
      <b-navbar-brand style="font-weight: bold; color: #45555F">
        <NuxtLogo />
        &nbsp;
        Mind Mapping In VR
      </b-navbar-brand>
    </b-navbar>
    <b-container class="app-container" fluid>
      <NuxtPage />
      <!-- Content here -->
      <b-card class="main-card mt-5 ml-auto mr-auto">
        <b-form-group
          label-cols-lg="3"
          label="Shipping Address"
          label-size="lg"
          label-class="font-weight-bold pt-0"
          class="mb-0"
        >
          <b-form-group
            label="Street:"
            label-for="nested-street"
            label-cols-sm="3"
            label-align-sm="right"
          >
            <b-form-input id="nested-street"></b-form-input>
          </b-form-group>
        
          <b-form-group
            label="City:"
            label-for="nested-city"
            label-cols-sm="3"
            label-align-sm="right"
          >
            <b-form-input id="nested-city"></b-form-input>
          </b-form-group>
        
          <b-form-group
            label="State:"
            label-for="nested-state"
            label-cols-sm="3"
            label-align-sm="right"
          >
            <b-form-input id="nested-state"></b-form-input>
          </b-form-group>
        
          <b-form-group
            label="Country:"
            label-for="nested-country"
            label-cols-sm="3"
            label-align-sm="right"
          >
            <b-form-input id="nested-country"></b-form-input>
          </b-form-group>
        
          <b-form-group
            v-slot="{ ariaDescribedby }"
            label="Ship via:"
            label-cols-sm="3"
            label-align-sm="right"
            class="mb-0"
          >
            <b-form-radio-group
              v-model="test"
              class="pt-2"
              :options="['Air', 'Courier', 'Mail']"
              :aria-describedby="ariaDescribedby"
            ></b-form-radio-group>
          </b-form-group>
        </b-form-group>
        <b-form-group>
          <h4 class="ml-1" style="font-weight: bold; color: #45555F">Insert information here</h4>
          <b-row class="mt-2">
            <b-col>
              <b-form-textarea
                id="textarea-large"
                v-model="prompt"
                size="lg"
                rows="8"
                placeholder="Large textarea"
              ></b-form-textarea>
            </b-col>
          </b-row>
        </b-form-group>
        <b-form-group>
          <h4 class="mt-2 ml-1" style="font-weight: bold; color: #45555F">Insert text file here</h4>
          <b-form-file accept=".txt"></b-form-file>
        </b-form-group>
        <div class="text-right">
          <b-button variant="secondary">Reset</b-button>
          <b-button variant="primary" @click="submit">Submit</b-button>
        </div>
        {{ rend }}
      </b-card>
    </b-container>
  </div>
</template>

<script>

export default {
  name: 'IndexPage',
  data() {
    return {
      rendering: process.server ? 'server' : 'client',
      test: '',
      prompt: ''
    }
  },
  computed: {
    rend() {
      return this.rendering
    }
  },
  methods: {
    async submit() {
      alert(this.test)
      await this.requestToAPI()
    },
    async requestToAPI() {

      console.log(this.prompt)
      const vals = await this.$axios.$post('/request', {
        val: this.prompt,
      }).then(res => {
        console.log('res')
        console.log(res)
      })
      .catch(err => {
        console.log('errrr')
        console.log(err)
      })
      console.log(vals)
      
    }
  }
}
</script>
<style>
body {
  background-color: #45555F;
}

.app-navbar {
  height: 6vh;
}

.main-card {
  width: 95vw;
  height: calc(100vh - 15vh);
}


</style>