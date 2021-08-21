<template>
  <div class="hello">
    <input type="text" v-model="eventID" />
    {{ apierrMessage }}
    <hr />
    <div class="event-list">{{ loginResponse ? 'Is Logged In' : 'Not Logged In' }}</div>

    <button @click="login">log me in</button>
    <hr />
    <button @click="getEventDetails">Get my event</button>

    <hr />
    <h1>Last request response</h1>
    <hr />
    <pre v-html="lastRequestResponse"></pre>
    <hr />
    <pre v-for="res in lastRequestResponse" :key="res.id">

  <span v-html="res"></span></pre>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',

  props: {
    msg: String
  },
  data() {
    return {
      info: null,
      loginResponse: null,
      authToken: null,
      eventID: null,
      eventResponse: null,
      apierrMessage: null,
      lastRequestResponse: []
    }
  },
  methods: {
    async login() {
      let payload = {
        AuthParameters: {
          USERNAME: 'ENTERUSERNAME',
          PASSWORD: 'ENTERPASSWORD!'
        },
        AuthFlow: 'USER_PASSWORD_AUTH',
        ClientId: 'ENTERUSERNAME'
      }

      let headers = {
        'Content-Type': 'application/x-amz-json-1.1'
      }
      let res = null

      await this.$http
        .post('https://api.magentacloudcapture.com/vendor/auth/login', payload, {
          headers
        })
        .then((response) => {
          console.log(response.data.AuthenticationResult)
          res = Object.assign({}, res, response.data)
          this.loginResponse = Object.assign({}, this.loginResponse, response.data)
        })

      this.setResponseData(res)
    },
    getEventDetails() {
      let myapiRoute = `https://api.magentacloudcapture.com/event/public/${this.eventID}`
      let accessToken = this.loginResponse.AuthenticationResult

      this.$http
        .get(myapiRoute, { headers: { Authorization: accessToken } })
        .then((response) => {
          console.dir(response.data)
          this.eventResponse = response.data
        })

      this.$http
        .get(myapiRoute)
        .then((response) => (this.totalVuePackages = response.data.total))
        .catch((error) => {
          this.errorMessage = error.message
          console.error('There was an error!', error)
        })
    },
    syntaxHighlight(json) {
      json = json.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
      return json.replace(
        /("(\\u[a-zA-Z0-9]{4}|\\[^u]|[^\\"])*"(\s*:)?|\b(true|false|null)\b|-?\d+(?:\.\d*)?(?:[eE][+\-]?\d+)?)/g,
        function (match) {
          var cls = 'number'
          if (/^"/.test(match)) {
            if (/:$/.test(match)) {
              cls = 'key'
            } else {
              cls = 'string'
            }
          } else if (/true|false/.test(match)) {
            cls = 'boolean'
          } else if (/null/.test(match)) {
            cls = 'null'
          }
          return '<span class="' + cls + '">' + match + '</span>'
        }
      )
    },
    setResponseData(data) {
      let resstring = JSON.stringify(data, undefined, 4)
      console.log(resstring)
      let formattedres = this.syntaxHighlight(resstring)

      this.lastRequestResponse.push(formattedres)
    }
  }
}
</script>
