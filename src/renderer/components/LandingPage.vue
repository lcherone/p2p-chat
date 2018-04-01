<template>
  <main>
    <div id="header">
      <h1>
        P2P Chat
        <span id="status">Status: {{ status.to ? status.to : 'connecting' }}</span>
      </h1>
    </div>
    <div id="messages">
      <div v-for="message in chats">
        {{ message.date }} {{ message.peer.id.substr(0, 10) }}: {{ message.msg }}
      </div>
    </div>
    <div id="footer">
      <input id="input" type="text" v-model="text" @keyup.enter="send_chat()" placeholder="Enter your message..."><button @click="send_chat()">Send</button>
    </div>
  </main>
</template>

<script>
  const p2p = require('p2p')

  export default {
    name: 'landing-page',
    components: { },
    data () {
      return {
        peer: {},
        peers: [],
        log: [],
        status: {},
        chats: [],
        text: ''
      }
    },
    methods: {
      send_chat () {
        this.broadcast('handle/chat', {
          peer: this.peer.self,
          msg: this.text,
          date: new Date()
        })
        this.text = ''
      },
      broadcast (run, value) {
        for (var peer in this.peers) {
          if (!this.peers.hasOwnProperty(peer)) continue
          this.peer.remote(this.peers[peer]).run(run, value, (err, result) => {
            if (err) {
              console.log(err)
            }
            console.log(result)
          })
        }
      }
    },
    mounted () {
      this.peer = p2p.peer({
        host: 'localhost',
        port: (Math.floor(Math.random() * 10) + 3000),
        wellKnownPeers: [
          { host: 'localhost', port: 3000 },
          { host: 'localhost', port: 3001 },
          { host: 'localhost', port: 3002 },
          { host: 'localhost', port: 3003 },
          { host: 'localhost', port: 3004 },
          { host: 'localhost', port: 3005 },
          { host: 'localhost', port: 3006 },
          { host: 'localhost', port: 3007 },
          { host: 'localhost', port: 3008 },
          { host: 'localhost', port: 3009 },
          { host: 'localhost', port: 3010 }
        ],
        serviceInterval: '1s'
      })

      this.peer.handle.chat = (payload, done) => {
        this.chats.push(payload)

        var container = this.$el.querySelector('#messages')
        container.scrollTop = container.scrollHeight
        done(null, true)
      }

      this.peer.on('environment::successor', successor => {
        this.log.push({ event: 'Changed successor', data: successor })
      })

      this.peer.on('environment::predecessor', predecessor => {
        this.log.push({ event: 'Changed predecessor', data: predecessor })
      })

      this.peer.on('status::*', status => {
        this.log.push({ event: 'Changed status', data: status })
        this.status = status
        this.peers = this.peer.wellKnownPeers.get()
      })

      this.peers = this.peer.wellKnownPeers.get()
    }
  }
</script>

<style>
  @import url('https://fonts.googleapis.com/css?family=Source+Sans+Pro');

  wrapper, body {
    margin: 0px;
    font-family: "Source Sans Pro";
    background: black;
    color: limegreen
  }

  #header {
    height: 40px;
    background: red
  }

  #messages {
    height: calc(100vh - 120px);
    overflow-y: scroll;
    padding: 10px;
    padding-bottom: 30px
  }

  #status {
    float: right;
    margin-right: 10px
  }

  #footer {
    height: 40px
  }

  #input {
    border: 0;
    width: calc(100vw - 100px);
    height: 19px;
    padding: 10px;
    background: black;
    color: limegreen
  }

  button {
    height: 40px;
    width: 65px;
    background: limegreen;
    border: 0;
    float: right
  }

  h1 {
    -webkit-margin-before: 0;
    -webkit-margin-after: 0;
    -webkit-margin-start: 0px;
    -webkit-margin-end: 0px;
    padding-left: 7px
  }

</style>