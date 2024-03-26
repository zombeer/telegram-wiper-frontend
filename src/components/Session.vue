<script>



export default {
  name: 'Session',
  data() {
    return {
      session_name: null,
      qr_code: null,
      status: null,
      auth_password: null,
      channel: null,
    }
  },
  created() {
    this.API_URL = 'http://localhost:8000/';
    this.WAITING_FOR_QR_LOGIN = "waiting_for_qr_login";
    this.WAITING_FOR_AUTH = "waiting_for_auth";
    this.WAITING_FOR_CHANNEL_SELECTION = "waiting_for_channel_selection";
  },
  methods: {
    async getSession() {
      try {
        const response = await fetch(this.API_URL)
        const data = await response.json()

        this.session_name = data.name
        this.qr_code = data.qr_code
        this.status = data.status

        } 
      catch (error) {
        console.error(error)
      }
    },
    async postSession() {
      try {
        const response = await fetch(this.API_URL, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ session_name: this.session_name })
        })
        
        if (response.status === 400) {
          this.session_name = null;
          await this.getSession();
        } else {
          const data = await response.json()
          this.qr_code = data.qr_code
          this.status = data.status
        }
      } 
      catch (error) {
        console.error(error)
      }
    },
    async postAuthPassword(){
      try {
        const response = await fetch(this.API_URL + 'auth', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({ session_name: this.session_name, password: this.auth_password })
        })
        const data = await response.json()
        this.status = data.status
      } 
      catch (error) {
        console.error(error)
      }
    },
    async pickChannel(){
      try {
        const response = await fetch(this.API_URL + 'clean_channel', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({session_name: this.session_name, channel: this.channel })
        })
        const data = await response.json()
        console.log('Cleaned channel')
        console.log(data)
      } 
      catch (error) {
        console.error(error)
      }
    }
},
  mounted() {
    if (localStorage.getItem('session_name')) {
      this.session_name = localStorage.getItem('session_name');
      this.postSession();
    } 
    else {
      this.getSession();
    }
  },
  watch: {
    session_name(newName) {
      localStorage.setItem('session_name', newName);
    }
  }
}
</script>

<template>
    <div>
        <!-- General info -->
        <h1>Session: {{ session_name }}</h1>
        <p>{{ status }}</p>

        <!-- QR code -->
        <img :src="qr_code" alt="QR Code" v-if="status == WAITING_FOR_QR_LOGIN"/>

        <!-- Password input -->
        <div v-if="status == WAITING_FOR_AUTH">
          <input v-model="auth_password" placeholder="Password" type="password"  />
          <button @click="postAuthPassword">Ok</button>
        </div>

        <!-- Channel selection -->
        <div v-if="status == WAITING_FOR_CHANNEL_SELECTION">
          <input v-model="channel" placeholder="Channel name or URL" />
          <button @click="pickChannel">Ok</button>
        </div>

      </div>  
</template>