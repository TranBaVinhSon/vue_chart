<template>
  <div class="content">
    <div class="container">
      <div class="Search__container">
        <!-- eslint-disable-next-line vue/valid-v-model -->
        <input
          class="Search__input"
          @keyup.enter="requestData"
          placeholder="npm package name"
          type="search"
          name="search"
          v-model="package"
        />
        <button class="Search__button" @click="requestData">Find</button>
      </div>
      <div class="error-message" v-if="showError">{{ errorMessage }}</div>
      <hr />
      <h1 class="title" v-if="loaded">{{ packageName }}</h1>
      <div class="Chart__container" v-if="loaded">
        <div class="Chart__title">
          Downloads per Day
          <span>{{ period }}</span>
          <hr />
        </div>
        <div class="Chart__content">
          <line-chart
            v-if="loaded"
            :chart-data="downloads"
            :chart-labels="labels"
          ></line-chart>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import axios from 'axios'
import LineChart from '@/components/LineChart'
export default {
  components: {
    LineChart
  },
  props: {},
  data () {
    return {
      package: '',
      packageName: '',
      period: 'last-month',
      loaded: false,
      downloads: [],
      labels: [],
      showError: false,
      errorMessage: 'Please enter a package name'
    }
  },
  mounted () {
    if (this.$route.params.package) {
      this.package = this.$route.params.package
      this.requestData()
    }
  },
  methods: {
    resetState () {
      this.loaded = false
      this.showError = false
    },
    requestData () {
      if (
        this.package === null ||
        this.package === '' ||
        this.package === 'undefined'
      ) {
        this.showError = true
        return
      }
      this.resetState()
      axios
        .get(
          `https://api.npmjs.org/downloads/range/${this.period}/${this.package}`
        )
        .then((response) => {
          console.log(response.data)
          this.downloads = response.data.downloads.map(
            (download) => download.downloads
          )
          this.labels = response.data.downloads.map((download) => download.day)
          this.packageName = response.data.package
          this.setURL()
          this.loaded = true
        })
        .catch((err) => {
          this.errorMessage = err.response.data.error
          this.showError = true
        })
    },
    setURL () {
      history.pushState(
        { info: `npm-stats ${this.package}` },
        this.package,
        `/#/${this.package}`
      )
    }
  }
}
</script>
