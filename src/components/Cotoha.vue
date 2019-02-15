<template>
  <div id="cotoha">
    <p>{{ text }}</p>
    <p>感情分析結果 : {{ emotions }}</p>
    <p>文タイプ分析結果 : {{ sentenceTypes }}</p>
    <textarea id="sentiment" :value="text" @change="inputChanged($event)"></textarea>
  </div>
</template>

<script>
'use strict'
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
        this.token = res.data['access_token']
      })
      .catch(error => {
        console.log(error)
      })
  },
  methods: {
    inputChanged (e) {
      this.text = e.target.value
      const headers = {
        'content-type': 'application/json',
        Authorization: `Bearer ${this.token}`
      }
      const data = { sentence: this.text }
      this.sentiment(headers, data)
      this.sentenceType(headers, data)
    },
    sentiment (headers, data) {
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
    },
    sentenceType (headers, data) {
      axios({
        method: 'POST',
        headers: headers,
        url: `${config.BASE_URL}/v1/sentence_type`,
        data: data
      })
        .then(res => {
          this.sentenceTypes = res.data['result']['dialog_act']
        })
        .catch(error => {
          console.log(error)
        })
    }
  }
}
</script>

<style scoped>
  textarea {
    width: 500px;
    height: 100px;
  }
</style>
