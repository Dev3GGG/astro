<template>
  <transition name="fade" appear>
    <div v-if="showLoader" class="page-loader">
      <img :src="Logo" alt="Nizana" class="loading" />
    </div>
  </transition>
</template>

<script setup lang="ts">
import { ref, watch, onMounted } from "vue"
import { useProgress } from "@tresjs/cientos"
import Logo from "../assets/nizana.svg?url"
import { gsap } from 'gsap'

const { hasFinishLoading } = await useProgress()
const showLoader = ref(true)

function announceReady() {
  window.dispatchEvent(new CustomEvent("tres:ready"))
  document.documentElement.classList.add("tres-ready")
}

watch(hasFinishLoading, (done) => {
  if (done) {
    setTimeout(() => {
      showLoader.value = false
      announceReady()
    }, 1000)
  }
})

onMounted(() => {
  const tl = gsap.timeline();
  tl.from(".loading", {
    opacity: 0,
    scaleX: 0.8,
    scaleY: 0.8,
    y: -25,
    duration: 1.2,
    delay: 0.1,
    ease: "back.out(1.7)",
  })
  if (hasFinishLoading.value) {
    showLoader.value = false
    announceReady()
  }
})
</script>

<style scoped>
.page-loader {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: grid;
  place-items: center;
  background: #ffffff;
}

.loading {
  width: clamp(266px, 22vw, 200px);
  height: auto;
  user-select: none;
  -webkit-user-drag: none;
  animation: logo-in 0.8s ease-out 0.15s both,
    slow-spin 12s linear infinite;
}

</style>
