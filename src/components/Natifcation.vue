<script setup>
import { ref, onUnmounted, watchEffect } from 'vue'
import { FontAwesomeIcon } from '@fortawesome/vue-fontawesome'
import { library } from '@fortawesome/fontawesome-svg-core' // Import library
import { faVuejs } from '@fortawesome/free-brands-svg-icons'
import { faBell } from '@fortawesome/free-solid-svg-icons'
import io from 'socket.io-client'
import axios from 'axios'

function sendNoti(name, author, detail) {
  const notification = new Notification(name, {
    body: `Öğretmen: ${author}\n\nDers detayı: ${detail}`,
    icon: 'https://o.remove.bg/downloads/7184f030-f48d-40a9-930c-8650af1cc25d/images-removebg-preview.png'
  })

  notification.onclick = function () {
    if (window.focus) {
      window.focus()
    }
  }
}

const active = ref(true)
const notiNum = ref(0)
const artcile = ref(null)
const min = ref(60000)
function timeConvertor(arg) {
  const timeDifference = new Date() - new Date(arg)
  const day = 86400000
  const hour = 3600000

  switch (true) {
    case timeDifference > day:
      return Math.floor(timeDifference / day) + ' gün önce'
    case timeDifference > hour:
      return Math.floor(timeDifference / hour) + ' saat önce'
    case timeDifference > min.value:
      return Math.floor(timeDifference / min.value) + ' dakika önce'
    default:
      return 'yeni'
  }
}
// console.log(timeConvertor('2023-11-02T10:44:02.039+00:00'))
watchEffect(() => {
  updateNoti()
  Notification.requestPermission().then((permission) => {
    if (permission === 'denied') {
      alert(
        'Notification permission denied. You can update your permission settings in your browser.'
      )
    } else if (permission === 'default') {
      alert('Notification permission prompt closed without making a selection.')
    }
  })
})
function toggle() {
  // console.log(artcile._rawValue[0].createdAt)
  const upAt = artcile._rawValue[0].createdAt
  active.value = !active.value
  localStorage.setItem('last_noti', upAt)
  notiNum.value = 0
}

function updateNoti() {
  axios
    .get('http://localhost:5000/api/v1/article')
    .then(function (response) {
      artcile.value = response.data.data.reverse()
      upNotiNum(response.data.data)
    })
    .catch(function (error) {
      console.log(error)
    })
}

function upNotiNum(arg) {
  const oldAt = new Date(localStorage.getItem('last_noti'));

  for (let i = 0; i < arg.length; i++) {
    const arrAt = new Date(arg[i].createdAt);

    if (Number(arrAt) > Number(oldAt)) {
      notiNum.value += 1; // Corrected this line
    }
  }
}



library.add(faVuejs, faBell)

const socket = io('http://localhost:5000')

socket.on('connect', () => {
  console.log('Connected to the server')
})

socket.on('dataChange', (change) => {
  console.log(change)
  const newUpdate = {
    _id: change.fullDocument._id,
    message: `New data change: ${change.fullDocument.id}`
  }
  sendNoti(change.fullDocument.name, change.fullDocument.author, change.fullDocument.details)
  // console.log(newUpdate)
  updateNoti()
})

socket.on('disconnect', () => {
  console.log('Disconnected from the server')
})

onUnmounted(() => {
  socket.disconnect()
})
</script>

<template>
  <div>
    <label @click="toggle">
      <font-awesome-icon :icon="['fas', 'bell']" style="color: blue; font-size: xx-large" />

      <span class="badge" v-if="active && notiNum">{{ notiNum }}</span>

      <div class="active" v-else-if="!active">
        <div
          style="
            text-align: end;
            border-radius: 5px;
            cursor: pointer;
            background-color: orange;
            padding: 5px 10px;
          "
        >
          X
        </div>
        <div v-for="todo in artcile" :key="todo.id">
          <h2 class="author">{{ todo.author }}</h2>
          <div
            style="display: flex; align-items: center; justify-content: space-between; color: brown"
          >
            <h6>{{ todo.name }}</h6>
            <h6>{{ timeConvertor(todo.createdAt) }}</h6>
            <h6>{{ todo.id }}</h6>
          </div>

          <br />
          <h3>Detay:</h3>
          <p class="detail">{{ todo.details }}</p>
          <br />
          <hr />

          <br />
        </div>
      </div>
    </label>
  </div>
</template>
<style>
label {
  position: relative;
  cursor: pointer;
  display: inline-block;
  margin: 20px 20px 10px 20px;
}
.author {
  font-weight: 900;
}
.badge {
  position: absolute;
  top: -10px;
  left: 14px;
  padding: 1px 3px;
  border-radius: 5px;
  background-color: #fff;
  color: #000;
  width: auto;
  font-weight: bolder;
  height: 25px;
}
.detail {
  background-color: #797373;
  color: #fff;
  border-radius: 5px;
  padding: 5px;
  text-align: justify;
}
.active {
  overflow-x: scroll;
  position: absolute;
  border-radius: 5px;
  padding: 10px;
  top: -10px;
  left: 14px;
  background-color: #fff;
  color: #000;
  animation: mymove 1s forwards;
  cursor: auto;
}

::-webkit-scrollbar {
  position: relative;
  width: 5px;
  top: 20px;
  margin: 20px;
  padding: 200px;
}

::-webkit-scrollbar-thumb {
  background-color: gray;
  top: 20px;
}

/* Hide horizontal scrollbar */
::-webkit-scrollbar-horizontal {
  height: 10px;
}

::-webkit-scrollbar-thumb:horizontal {
  background-color: grey;
}

@keyframes mymove {
  0% {
    width: 0px;
    height: 0px;
  }
  100% {
    width: 350px;
    height: 300px;
  }
}
</style>
