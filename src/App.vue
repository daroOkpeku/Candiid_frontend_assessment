<script setup>
import { RouterView } from 'vue-router'
import Navbar from './components/Navbar.vue'
import './assets/main.css'
</script>

<template>
  <Navbar />
  <div>
    <RouterView />
  </div>
</template>

<style scoped></style>
