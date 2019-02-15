<template>
  <div id="cotoha">
    <p>{{ text }}</p>
    <p>感情分析結果 : {{ emotions }}</p>
    <input id="sentiment" type="text" :value="text" @change="sentiment($event)">
  </div>
</template>

<script>
'use strict';
import axios from 'axios'
import config from '@/env/config'

export default {
  name: 'Cotoha',
  data () {
    return {
      text: '',
      emotions: [],
      sentenceTypes: [],
      token: ''
    }
  },
  created () {
    const data = {
      grantType: 'client_credentials',
      clientId: config.CLIENT_ID,
      clientSecret: config.CLIENT_SECRET
    }
    axios.post(config.TOKEN_URL, data)
      .then(res => {
        console.log(res.data)
        this.token = res.data['access_token']
      })
      .catch(error => {
        console.log(error)
      })
  },
  methods: {
    sentiment (e) {
      const headers = {
        'content-type': 'application/json',
        Authorization: `Bearer ${this.token}`
      }
      const data = { sentence: e.target.value }
      this.text = e.target.value
      axios({
        method: 'POST',
        headers: headers,
        url: `${config.BASE_URL}/v1/sentiment`,
        data: data
      })
        .then(res => {
          this.emotions = res.data['result']['emotional_phrase'].flatMap(emoPhrase => emoPhrase['emotion'].split(','))
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>

<style scoped>

</style>
