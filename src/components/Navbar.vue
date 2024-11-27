<script lang="js" setup>
import { onMounted, ref } from 'vue'
import { useRouter } from 'vue-router'
import axios from 'axios'
import { AES, enc } from 'crypto-js'
const localData = ref({})

const router = useRouter()
const handleLink = (data) => {
  router.push({ name: data })
}

const handleLogout = () => {
  let headers = {
    'Content-Type': 'application/json',
    Authorization: 'Bearer ' + localData.value.token,
  }
  axios
    .get('http://127.0.0.1:8000/api/logout', {
      headers,
      withCredentials: true,
    })
    .then((response) => {
      if (response.data.success) {
        localStorage.removeItem('userinfo')
        setTimeout(() => {
          router.push({ name: 'login' })
        }, 1000)
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
    } catch (e) {
      console.error('Error parsing decrypted data:', e)
      localData.value = {} // Fallback to an empty object in case of parsing error
    }
  } else {
    localData.value = {} // Fallback if no data is in localStorage
  }
})
</script>
<template>
  <div class="w-full flex items-center justify-center py-3 border border-b-2">
    <ul class="w-1/4 flex flex-row items-center justify-center gap-3 m-auto">
      <li class="text-center capitalize cursor-pointer" @click="handleLink('home')">home</li>
      <li class="text-center capitalize cursor-pointer" @click="handleLink('view')">view</li>
      <li
        v-if="localData && Object.keys(localData).length > 0"
        class="text-center capitalize cursor-pointer"
        @click="handleLogout()"
      >
        logout
      </li>
    </ul>
  </div>
</template>
<style scoped></style>
