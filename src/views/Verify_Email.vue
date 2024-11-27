<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
const isconfirm = ref(false)

const router = useRouter()

let headers = {
  'Content-Type': 'application/json',
  Accept: 'application/json',
}
const parsedUrl = window.location.href
const email = new URL(parsedUrl).searchParams.get('email')
const code = new URL(parsedUrl).searchParams.get('code')
console.log(email, code)
if (email && code) {
  axios
    .get('http://127.0.0.1:8000/api/verify_email?email=' + email + '&code=' + code, {
      headers,
    })
    .then((response) => {
      console.log(response.data)
      if (response.data.success) {
        isconfirm.value = true
        setTimeout(() => {
          router.push({ name: 'login' })
        }, 1500)
      }
    })
}
</script>
<template>
  <div class="w-full flex items-center justify-center">
    <div class="w-2/5 m-auto h-48 rounded-md flex flex-col items-center mt-28 gap-6">
      <h2 class="text-2xl text-center text-black">Please Wait For email to be Verified</h2>
      <button class="text-lg bg-blue-500 rounded-md text-white px-2 py-2" v-if="isconfirm">
        confirm
      </button>
      <button class="text-lg bg-blue-500 rounded-md text-white px-2 py-2" v-else>
        Please Wait..
      </button>
    </div>
  </div>
</template>
<style scroped></style>
