<template>
  <div>
    <div v-if="spinner" class="flex fixed justify-center items-center bottom-0 top-0 left-0 right-0 bg-black bg-opacity-30">
      <div class="loader ease-linear rounded-full border-8 border-t-8 border-gray-200 h-32 w-32"></div>
    </div>
    <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-12">
      <div class="lg:text-center">
        <a class="text-base text-indigo-600 font-semibold tracking-wide uppercase" href="https://kicktemp.com" target="_blank ">by Kicktemp</a>
        <p class="mt-2 text-3xl leading-8 font-extrabold tracking-tight text-gray-900 sm:text-4xl">
          YOOtheme Pro Presets Downloader
        </p>
        <p class="mt-4 max-w-2xl text-xl text-gray-500 lg:mx-auto">
        </p>
        <div v-if="ratelimit != null && ratelimit.remaining < (ratelimit.limit - 40)" class="mt-4 max-w-2xl text-xl lg:mx-auto text-red-500">
          Request Limit {{ ratelimit.limit }}/{{ ratelimit.remaining }} - Reset {{ ratelimit.resetdate }}
        </div>
      </div>
      <div class=" md:col-span-2 mt-10">
        <div class="shadow sm:rounded-md sm:overflow-hidden">
          <div class=" bg-white p-6">
            <div class="block text-sm font-medium text-gray-700">
              GitHub
            </div>
            <div class="grid grid-cols-4 gap-6 mt-0">
              <div class="col-span-4 sm:col-span-1">
                <div class="mt-1 flex rounded-md shadow-sm">
                    <span
                        class="inline-flex items-center px-3 rounded-l-md border border-r-0 border-gray-300 bg-gray-50 text-gray-500 text-sm">
                      owner
                    </span>
                  <input type="text" id="github-owser" placeholder="nielsnuebel"
                         v-model="owner"
                         class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block w-full rounded-none rounded-r-md sm:text-sm border-gray-300"/>
                </div>
              </div>
              <div class="col-span-4 sm:col-span-1">
                <div class="mt-1 flex rounded-md shadow-sm">
                    <span
                        class="inline-flex items-center px-3 rounded-l-md border border-r-0 border-gray-300 bg-gray-50 text-gray-500 text-sm">
                      repo
                    </span>
                  <input type="text" id="github-repo" placeholder="lighthouse"
                         v-model="repo"
                         class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block w-full rounded-none rounded-r-md sm:text-sm border-gray-300"/>
                </div>
              </div>
              <div class="col-span-4 sm:col-span-1">
                <div class="mt-1 flex rounded-md shadow-sm">
                    <span
                        class="inline-flex items-center px-3 rounded-l-md border border-r-0 border-gray-300 bg-gray-50 text-gray-500 text-sm">
                      branch
                    </span>
                  <input type="text" id="github-branch" placeholder="main"
                         v-model="branch"
                         class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block w-full rounded-none rounded-r-md sm:text-sm border-gray-300"/>
                </div>
              </div>
              <div class="col-span-4 sm:col-span-1">
                <div class="mt-1 flex rounded-md shadow-sm">
                    <span
                        class="inline-flex items-center px-3 rounded-l-md border border-r-0 border-gray-300 bg-gray-50 text-gray-500 text-sm">
                      path
                    </span>
                  <input type="text" id="github-path" placeholder="/"
                         v-model="path"
                         class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block w-full rounded-none rounded-r-md sm:text-sm border-gray-300"/>
                </div>
              </div>
            </div>

            <div class="mt-6 flex rounded-md shadow-sm">
              <span
                  class="inline-flex items-center px-3 rounded-l-md border border-r-0 border-gray-300 bg-gray-50 text-gray-500 text-sm">
                GitHub Repo
              </span>
              <input type="text" readonly v-model="githubrepo"
                     class="focus:ring-indigo-500 focus:border-indigo-500 flex-1 block w-full rounded-none sm:text-sm border-gray-300 bg-gray-50 cursor-not-allowed"/>
              <a :href="githubrepo" target="_blank"
                 class="inline-flex items-center px-3 rounded-r-md border border-l-0 rounded-none rounded-r-md border-gray-300 bg-gray-50 text-gray-500 text-sm">
                <ArrowTopRightOnSquareIcon class="h-5 w-5 text-gray-500"/>
              </a>
            </div>

            <div v-if="errormessage != null" class="text-red-500 mt-6">{{ errormessage }}</div>
            <div v-if="elements != null && elements.length > 0" class="mt-6">
              <label class="block text-sm font-medium text-gray-700">
                Elements
              </label>
              <div class="mt-1">
                <select @change="fetchPresets($event)" class="mt-1 block w-full py-2 px-3 border border-gray-300 bg-white rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
                  <option>Element auswählen</option>
                  <option v-for="element in elements" :key="element.name" :value="element.url">{{ element.name }}</option>
                </select>
              </div>
            </div>

            <div v-if="jsons != null && jsons.length > 0" class="mt-6">
              <label class="block text-sm font-medium text-gray-700">
                Presets
              </label>
              <div class="mt-1">
                <select v-model="preset" class="mt-1 block w-full py-2 px-3 border border-gray-300 bg-white rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
                  <option>Preset auswählen</option>
                  <option v-for="json in jsons.filter((json) => json.name.substring(json.name.length - 5) == '.json')" :key="json.name" :value="json">{{ json.name }}</option>
                </select>
              </div>
            </div>
          </div>
          <div v-if="showButton" class="px-4 py-3 bg-gray-50 text-center sm:px-6">
            <button @click="fetchFiles"
                    class="inline-flex justify-center py-2 px-6 border border-transparent shadow-sm text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500">
              Get Elements
            </button>
          </div>
          <div v-if="preset" class="px-4 py-3 bg-gray-50 text-center sm:px-6">
            <a target="_blank" :href="preset.download_url" class="inline-flex justify-center py-2 px-6 border border-transparent shadow-sm text-sm font-medium rounded-md text-white bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500" download>
              Download Preset {{ preset.name }}
            </a>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import moment from 'moment'
import { ArrowTopRightOnSquareIcon } from '@heroicons/vue/24/solid'

export default {
  name: 'App',
  components: {
    ArrowTopRightOnSquareIcon
  },
  data: () => {
    return {
      ratelimit: null,
      jsons: null,
      fetchTime: null,
      elements: null,
      element: null,
      preset: null,
      spinner: false,
      showButton: true,
      errormessage: null,
      owner: 'Kicktemp',
      repo: 'yootheme_presets',
      branch: 'main',
      path: '',
      githubrepo: ''
    }
  },
  created () {
    moment.locale('de-DE')
    const uri = window.location.search.substring(1)
    const params = new URLSearchParams(uri)
    if (params.get('owner') !== null) {
      this.owner = params.get('owner')
    }
    if (params.get('repo') !== null) {
      this.repo = params.get('repo')
    }
    if (params.get('path') !== null) {
      this.path = params.get('path')
    }
    if (params.get('branch') !== null) {
      this.branch = params.get('branch')
    }

    this.changeRepo()
    this.spinner = true
    this.errormessage = null
    this.jsons = null
    this.element = null
    this.elements = null

    fetch('https://api.github.com/rate_limit', {
      method: 'GET',
      headers: new Headers({
        'Content-Type': 'application/json',
        Accept: 'application/vnd.github.v3+json'
      })
    })
        .then(r => r.json())
        .then(json => {
          json.rate.resetdate = moment(new Date(json.rate.reset * 1000)).format('HH:mm DD.MM.YYYY')
          this.ratelimit = json.rate
          this.spinner = false
        })
  },
  watch: {
    owner: function (newVal, oldVal) {
      if (newVal !== oldVal && newVal !== '') {
        this.showButton = true
      }
      this.changeRepo()
    },
    repo: function (newVal, oldVal) {
      if (newVal !== oldVal && newVal !== '') {
        this.showButton = true
      }
      this.changeRepo()
    },
    path: function (newVal, oldVal) {
      if (newVal !== oldVal && newVal !== '') {
        this.showButton = true
      }
      this.changeRepo()
    },
    branch: function (newVal, oldVal) {
      if (newVal !== oldVal && newVal !== '') {
        this.showButton = true
      }
      this.changeRepo()
    }
  },
  methods: {
    changeRepo () {
      this.githubrepo = 'https://github.com/'
      if (this.owner !== '') {
        this.githubrepo = this.githubrepo + this.owner + '/'
      }
      if (this.repo !== '') {
        this.githubrepo = this.githubrepo + this.repo + '/'
      }
      if (this.branch !== '') {
        this.githubrepo = this.githubrepo + 'tree/' + this.branch + '/'
      }
      if (this.path !== '') {
        this.githubrepo = this.githubrepo + this.path
      }
    },
    fetchFiles () {
      this.jsons = null
      this.elements = null
      this.spinner = true
      this.errormessage = null
      let filesurl = 'https://api.github.com/repos/' + this.owner + '/' + this.repo + '/contents/' + this.path
      if (this.branch !== '' && this.branch !== 'main') {
        filesurl = filesurl + '?ref=' + this.branch
      }
      fetch(filesurl, {
        method: 'GET',
        headers: new Headers({
          'Content-Type': 'application/json',
          Accept: 'application/vnd.github.v3+json'
        })
      })
          .then((response) => {
            if (!response.ok) {
              throw new Error('HTTP status ' + response.status)
            }
            return response.json()
          })
          .then(json => {
            this.elements = json.filter(item => item.type === 'dir')
            this.files = json
            this.spinner = false
            this.showButton = false
          })
          .catch((error) => {
            this.errormessage = error
            this.spinner = false
            this.showButton = true
          })
    },
    fetchPresets (element) {
      this.jsons = null
      this.spinner = true
      this.errormessage = null
      fetch(element.target.value, {
        method: 'GET',
        headers: new Headers({
          'Content-Type': 'application/json',
          Accept: 'application/vnd.github.v3+json'
        })
      })
          .then((response) => {
            if (!response.ok) {
              throw new Error('HTTP status ' + response.status)
            }
            return response.json()
          })
          .then(json => {
            this.jsons = json.filter(item => item.type === 'file')
            this.spinner = false
            this.showButton = false
          })
          .catch((error) => {
            this.errormessage = error
            this.spinner = false
            this.showButton = true
          })
    }
  }
}
</script>

<style>
.loader {
  border-top-color: #3498db !important;
  -webkit-animation: spinner 1.5s linear infinite;
  animation: spinner 1.5s linear infinite;
}

@-webkit-keyframes spinner {
  0% {
    -webkit-transform: rotate(0deg);
  }
  100% {
    -webkit-transform: rotate(360deg);
  }
}

@keyframes spinner {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>