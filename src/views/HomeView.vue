<script setup>
import { onMounted, ref } from 'vue'
import VueDatePicker from '@vuepic/vue-datepicker'
import '@vuepic/vue-datepicker/dist/main.css'
import axios from 'axios'
import { AES, enc } from 'crypto-js'
import { useRouter } from 'vue-router'
const localData = ref({})
const router = useRouter()
const date = ref()
const title = ref('')
const description = ref('')
const status = ref('')
const message = ref('')
const isprocessing = ref(false)
const isEdit = ref(false)

const handleSubmit = () => {
  isprocessing.value = true
  let headers = {
    'Content-Type': 'application/json',
    Authorization: 'Bearer ' + localData.value.token,
  }
  let formdata = new FormData()
  formdata.append('title', title.value)
  formdata.append('description', description.value)
  formdata.append('status', status.value)
  formdata.append('due_date', date.value)
  axios
    .post('http://127.0.0.1:8000/api/create_task', formdata, { headers, withCredentials: true })
    .then((response) => {
      console.log(response.data)
      if (response.data.success) {
        isprocessing.value = false
        message.value = response.data.success
        setTimeout(() => {
          router.push({ name: 'view' })
        }, 1500)
      }
    })
    .catch((err) => {
      let error = err.response.data.errors
      if (error.title) {
        isprocessing.value = false
        message.value = error.title[0]
      } else if (error.description) {
        isprocessing.value = false
        message.value = error.description[0]
      } else if (error.status) {
        isprocessing.value = false
        message.value = error.status[0]
      } else if (error.date) {
        isprocessing.value = false
        message.value = error.date[0]
      }
    })
}

const parsedUrl = window.location.href
const page = new URL(parsedUrl).searchParams.get('id')

const handleEdit = () => {
  isprocessing.value = true
  let headers = {
    'Content-Type': 'application/json',
    Authorization: 'Bearer ' + localData.value.token,
  }
  let formdata = new FormData()
  formdata.append('id', parseInt(page))
  formdata.append('title', title.value)
  formdata.append('description', description.value)
  formdata.append('status', status.value)
  formdata.append('due_date', date.value)
  axios
    .put('http://127.0.0.1:8000/api/edit_task', formdata, { headers, withCredentials: true })
    .then((response) => {
      console.log(response.data)
      if (response.data.success) {
        isprocessing.value = false
        message.value = response.data.success
        setTimeout(() => {
          router.push({ name: 'view' })
        }, 1500)
      }
    })
    .catch((err) => {
      let error = err.response.data.errors
      if (error.title) {
        isprocessing.value = false
        message.value = error.title[0]
      } else if (error.description) {
        isprocessing.value = false
        message.value = error.description[0]
      } else if (error.status) {
        isprocessing.value = false
        message.value = error.status[0]
      } else if (error.date) {
        isprocessing.value = false
        message.value = error.date[0]
      }
    })
}

onMounted(() => {
  // Check if window and localStorage are available

  const userInfo = localStorage.getItem('userinfo')
  console.log(userInfo)
  if (userInfo) {
    // Decrypt and parse the user info
    const decryptedData = AES.decrypt(userInfo, 'HEALTH').toString(enc.Utf8)
    console.log(decryptedData)
    try {
      // Parse and assign the decrypted data to localData
      localData.value = JSON.parse(decryptedData)
      console.log(localData.value.token)
      let headers = {
        'Content-Type': 'application/json',
        Authorization: 'Bearer ' + localData.value.token,
      }
      if (page) {
        axios
          .get('http://127.0.0.1:8000/api/single_task?id=' + parseInt(page), {
            headers,
            withCredentials: true,
          })
          .then((response) => {
            console.log(response.data)
            if (response.data.success) {
              isEdit.value = true
              title.value = response.data.success.title
              description.value = response.data.success.description

              status.value = response.data.success.status
              date.value = new Date(response.data.success.due_date)
            }
          })
      }
    } catch (e) {
      console.error('Error parsing decrypted data:', e)
      localData.value = {}
      setTimeout(() => {
        router.push({ name: 'login' })
      }, 1500)
    }
  } else {
    localData.value = {} // Fallback if no data is in localStorage
  }
})
</script>

<template>
  <div class="w-full flex items-center justify-center">
    <section class="w-11/12 flex flex-col items-center justify-center gap-4 mt-8">
      <div class="w-3/4 flex flex-col items-center text-lg text-black" v-if="message">
        {{ message }}
      </div>
      <div class="w-3/4 flex flex-col items-center" v-else></div>
      <div class="w-3/4 flex flex-col items-center">
        <article class="w-11/12 flex flex-col items-center">
          <section class="w-full text-left text-lg capitalize">Title</section>
          <article class="w-full">
            <input type="text" v-model="title" class="w-full p-2 rounded-md border" />
          </article>
        </article>
      </div>

      <div class="w-3/4 flex flex-col items-center">
        <article class="w-11/12 flex flex-col items-center">
          <section class="w-full text-left text-lg capitalize">Description</section>
          <article class="w-full">
            <textarea
              type="text"
              v-model="description"
              rows="4"
              cols="8"
              class="w-full p-2 rounded-md border"
            >
            </textarea>
          </article>
        </article>
      </div>

      <div class="w-3/4 flex flex-col items-center">
        <article class="w-11/12 flex flex-col items-center">
          <section class="w-full text-left text-lg capitalize">Status</section>
          <article class="w-full">
            <select class="w-full py-2" v-model="status">
              <option value="Select Status">Select Status</option>
              <option value="Pending">Pending</option>
              <option value="In Progress">In Progress</option>
              <option value="Completed">Completed</option>
            </select>
          </article>
        </article>
      </div>

      <div class="w-3/4 flex flex-col items-center">
        <article class="w-11/12 flex flex-col items-center">
          <section class="w-full text-left text-lg capitalize">Due Date</section>
          <article class="w-full">
            <VueDatePicker v-model="date"></VueDatePicker>
          </article>
        </article>
      </div>

      <div class="w-3/4 flex flex-col items-center">
        <article class="w-11/12 flex flex-col items-center">
          <button
            v-if="isEdit"
            class="w-full py-2 rounded-md bg-blue-600 text-white text-center flex items-center justify-center"
            @click="handleEdit"
          >
            <span v-if="isprocessing">{{ 'Please wait...' }}</span>
            <span v-else>{{ 'Edit' }}</span>
          </button>

          <button
            v-else
            class="w-full py-2 rounded-md bg-blue-600 text-white text-center flex items-center justify-center"
            @click="handleSubmit"
          >
            <span v-if="isprocessing">{{ 'Please wait...' }}</span>
            <span v-else>{{ 'Submit' }}</span>
          </button>
        </article>
      </div>
    </section>
  </div>
</template>
