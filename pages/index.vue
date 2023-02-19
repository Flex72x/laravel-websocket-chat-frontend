<template>
  <div class="block-chat">
    <div id="statusConnection" class="px-3 d-flex flex-row align-items-center">
      <div class="status" :class="isConnected ? 'bg-success' : 'bg-danger'"></div>
      <span class="ms-1">{{isConnected ? 'Подключено' : 'Не подключено'}}</span>
    </div>

    <div class="messages-list p-3 d-flex flex-column flex-wrap" ref="messagesList">
      <div
          v-for="message in messages"
          :key="message.id"
          class="px-3 py-1 rounded-3 w-auto my-1 text-break mw-75"
          :class="message.userId === userId ? 'text-bg-secondary ms-auto ' : 'text-bg-dark me-auto'"
      >
        <span>{{message.message}}</span>
        <div class="text-white text-end fs-6">{{new Date(message.date).toLocaleTimeString().slice(0,-3)}}</div>
      </div>
    </div>

    <div class="send-message-block">
      <form @submit.prevent="sendMessage">
        <div class="input-group px-2 mb-2">
          <input v-model="message" type="text" class="form-control rounded-5 rounded-end" placeholder="Введите сообщение">
          <button :disabled="!message.length" type="submit" class="btn btn-dark rounded-5 rounded-start">
            Отправить
          </button>
        </div>
      </form>
    </div>
  </div>
</template>

<script setup>

import {v4} from "uuid";

const messages = ref([])
const message = useState('message', () => '')
const userId = v4()
const isConnected = ref(false)

const messagesList = ref(null)

let socket

if (process.client) {
  socket = new WebSocket('ws://192.168.1.73:8090')

  socket.onopen = () => {
    isConnected.value = true
  }

  socket.onmessage = (event) => {
    messages.value = [...messages.value, JSON.parse(event.data)]
  }

  socket.onclose = () => {
    isConnected.value = false
  }
}

const sendMessage = () => {
  const dataMessage = {
    id: v4(),
    userId: userId,
    message: message.value,
    date: new Date()
  }
  socket.send(JSON.stringify(dataMessage))

  messages.value = [...messages.value, dataMessage]

  message.value = ''
}
</script>

<style scoped>
.block-chat {
  min-height: 100vh;
  display: grid;
  grid-template-areas:
  "s"
  "t"
  "b";
  grid-template-rows: auto 1fr auto;
}

.messages-list {
  grid-area: t;
}

.send-message-block {
  grid-area: b;
}

#statusConnection {
  grid-area: s;
}

.status {
  width: 10px;
  height: 10px;
  border-radius: 50%;
}
.mw-75 {
  max-width: 75%;
}
</style>