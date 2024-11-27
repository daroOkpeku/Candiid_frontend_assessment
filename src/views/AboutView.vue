<script setup>
import { onMounted, ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'
import { AES, enc } from 'crypto-js'
const data = ref([])
const pagdata = ref()
const first = ref()
const second = ref()
const router = useRouter()
const localData = ref({})

const editTask = (id) => {
  console.log(id)

  router.push({ name: 'home', query: { id: id } })
}

const deleteTask = (id) => {
  let headers = {
    'Content-Type': 'application/json',
    Authorization: 'Bearer ' + localData.value.token,
  }

  axios
    .delete('http://127.0.0.1:8000/api/delete_task', {
      data: { id: id },
      headers,
    })
    .then((response) => {
      console.log(response.data)
      if (response.data.data) {
        data.value = response.data.data
      }
    })
    .catch((error) => {
      console.error(error.response?.data || error.message)
    })
}

const handleNextpag = (link) => {
  if (link != null) {
    let headers = {
      'Content-Type': 'application/json',
      Authorization: 'Bearer ' + localData.value.token,
    }
    axios.get(link, { headers, withCredentials: true }).then((response) => {
      console.log(response.data)
      if (response.data.data) {
        pagdata.value = response.data.links
        data.value = response.data.data

        const parsedUrl = response.data.links.first
        const page = new URL(parsedUrl).searchParams.get('page')
        first.value = page

        const parsedUrlx = response.data.links.last
        const pagesecond = new URL(parsedUrlx).searchParams.get('page')
        second.value = pagesecond
      }
    })
  }
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

      axios
        .get('http://127.0.0.1:8000/api/alltask?page=1', { headers, withCredentials: true })
        .then((response) => {
          console.log(response.data)
          if (response.data.data) {
            pagdata.value = response.data.links
            data.value = response.data.data

            const parsedUrl = response.data.links.first
            const page = new URL(parsedUrl).searchParams.get('page')
            first.value = page

            const parsedUrlx = response.data.links.last
            const pagesecond = new URL(parsedUrlx).searchParams.get('page')
            second.value = pagesecond
          }
        })
    } catch (e) {
      console.error('Error parsing decrypted data:', e)
      localData.value = {} // Fallback to an empty object in case of parsing error
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
  <div class="flex flex-col items-center justify-center">
    <article class="w-full overflow-x-auto flex flex-col items-center">
      <section class="w-[90%] sm:w-[90%] md:w-[90%] lg:w-[90%] border m-auto mt-10">
        <table class="w-full border py-2 rounded-md">
          <thead>
            <tr>
              <th class="font-bold capitalize bg-blue-500 text-white text-center py-2">Title</th>
              <th class="font-bold capitalize bg-blue-500 text-white text-center py-2">Status</th>
              <th class="font-bold capitalize bg-blue-500 text-white text-center py-2">
                Description
              </th>
              <th class="font-bold capitalize bg-blue-500 text-white text-center py-2">Due Date</th>
              <th class="font-bold capitalize bg-blue-500 text-white text-center py-2">action</th>
            </tr>
          </thead>
          <tbody class="covertable">
            <tr v-for="(dat, index) in data" :key="index">
              <td class="text-sm font-medium capitalize text-center py-2">{{ dat.title }}</td>
              <td class="text-sm font-medium capitalize text-center py-2">{{ dat.status }}</td>
              <td class="text-sm font-medium capitalize text-center py-2">
                {{ dat.description ? dat.description.substring(0, 30) + '...' : '' }}
              </td>
              <td class="text-sm font-medium capitalize text-center py-2">
                {{
                  new Date(dat.due_date).toLocaleDateString('en-US', {
                    year: 'numeric',
                    month: '2-digit',
                    day: '2-digit',
                  })
                }}
              </td>
              <td class="text-sm font-medium capitalize text-center py-2">
                <div
                  class="w-1/3 m-auto flex flex-col gap-2 sm:flex sm:flex-col md:flex md:flex-row md:space-x-2 lg:flex lg:flex-row lg:space-x-2 items-center"
                >
                  <a
                    class="py-2 px-2 rounded-md bg-green-400 text-white text-center cursor-pointer"
                    @click="editTask(dat.id)"
                    >Edit</a
                  >
                  <a
                    class="py-2 px-2 rounded-md bg-red-400 text-white text-center cursor-pointer"
                    @click="deleteTask(dat.id)"
                    >Delete</a
                  >
                </div>
              </td>
            </tr>
          </tbody>
        </table>
      </section>
    </article>

    <section class="w-full flex items-center justify-center">
      <div v-if="pagdata" class="w-1/3 flex flex-row items-center justify-center gap-3 m-auto">
        <button @click="handleNextpag(pagdata.prev)">Prev</button>
        <button v-if="first" @click="handleNextpag(pagdata.first)">{{ first }}</button>
        <button v-if="second != 1" @click="handleNextpag(pagdata.last)">{{ second }}</button>
        <button @click="handleNextpag(pagdata.next)">next</button>
      </div>
    </section>
  </div>
</template>

<style></style>
