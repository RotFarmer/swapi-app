<script setup lang="ts">
import { RouterLink, RouterView } from 'vue-router'
import { ref } from 'vue'
import { useDark, useToggle } from "@vueuse/core";
import '@fortawesome/fontawesome-free/css/all.css'
import { initializeApp } from "firebase/app";
import { getAnalytics } from "firebase/analytics";

const firebaseConfig = {
  apiKey: "AIzaSyDxpQnQfBSBjeSBbZ4xXhOUeukvdOaTWAg",
  authDomain: "starwars-api-be620.firebaseapp.com",
  projectId: "starwars-api-be620",
  storageBucket: "starwars-api-be620.appspot.com",
  messagingSenderId: "731188515950",
  appId: "1:731188515950:web:501c279e8719a8bd81027f",
  measurementId: "G-5WZMG4LWPV"
};

const app = initializeApp(firebaseConfig);
const analytics = getAnalytics(app);
const isDark = useDark()
const toggleDark = useToggle(isDark)

</script>

<template>
  <div class="wrapper" :class="{ light: !isDark, dark: isDark }">
    <header>
      <nav>
        <button class="color-button" @click="$router.push('/')">Home</button>
        <button class="color-button" @click="toggleDark()">
          <i class="fas fa-sun" v-if="!isDark"></i>
          <i class="fas fa-moon" v-else></i>
          <span class="toggle-text"> {{ isDark ? 'Dark' : 'Light' }}</span>
        </button>
      </nav>
    </header>

    <main>
      <RouterView />
    </main>
  </div>
</template>

<style scoped>
.wrapper {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  min-width: 65vw;
  align-items: center;
  justify-content: center;
  height: 100%;
  width: 100%;
}

header {
  line-height: 1.5;
  padding: 1rem;
  width: 50%;
}

nav {
  width: 100%;
  font-size: 12px;
  text-align: center;
  margin-top: 1rem;
  display: flex;
  justify-content: space-between;
}


main {
  flex: 1;
  padding: 1rem;
  display: flex;
}

.color-button {
  margin-right: 0.5rem;
  padding: 0.25rem 0.5rem;
  font-size: 1rem;
  background-color: transparent;
  cursor: pointer;
  border: none;
  color: rgb(119, 133, 133);
}

.color-button:hover {
  background-color: #333;
  color: #fff;
  border-radius: 0.25rem;
}

/* Define styles for the dark mode */
.dark-mode {
  background-color: #111;
  color: #fff;
}

.dark-mode nav a {
  border-color: #999;
  color: #bbb;
}

.dark-mode nav a.router-link-exact-active {
  color: #fff;
}

.dark-mode .color-button {
  background-color: #333;
  color: #fff;
}

.dark {
  background: #16171d;
  color: #fff;
}

.toggle-text {
  margin-left: 0.5rem;
}
</style>
