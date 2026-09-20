<template>
  <Transition name="fade" appear>
    <div v-if="showLoader" class="page-loader">
      <div class="logo-spinner">
        <img
          ref="logoElement"
          :src="Logo"
          alt="Nizana"
          class="loading"
        />
      </div>
    </div>
  </Transition>
</template>

<script setup lang="ts">
import { onBeforeUnmount, onMounted, ref } from "vue"
import { gsap } from "gsap"
import Logo from "../assets/nizana.svg?url"

const showLoader = ref(true)
const logoElement = ref<HTMLImageElement | null>(null)

let logoAnimation: gsap.core.Tween
let loaderTimer: ReturnType<typeof setTimeout>

onMounted(() => {
  logoAnimation = gsap.from(logoElement.value, {
    opacity: 0,
    scaleX: 0.8,
    scaleY: 0.8,
    y: -25,
    duration: 1.2,
    delay: 0.1,
    ease: "back.out(1.7)",
  })

  loaderTimer = setTimeout(() => {
    showLoader.value = false
  }, 1800)
})

onBeforeUnmount(() => {
  logoAnimation?.kill()
  clearTimeout(loaderTimer)
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

.logo-spinner {
  animation: slow-spin 12s linear infinite;
}

.loading {
  display: block;
  width: clamp(160px, 22vw, 266px);
  height: auto;
  user-select: none;
  pointer-events: none;
  -webkit-user-drag: none;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.4s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

@keyframes slow-spin {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}

@media (prefers-reduced-motion: reduce) {
  .logo-spinner {
    animation: none;
  }

  .fade-enter-active,
  .fade-leave-active {
    transition: none;
  }
}
</style>