<template>
  <div v-if="isVisible" class="page-loader" ref="loaderRef">
    <img src="/nizana.svg" alt="Nizana" class="loading" ref="logoRef" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { gsap } from 'gsap';

const isVisible = ref(true);
const loaderRef = ref<HTMLElement | null>(null);
const logoRef = ref<HTMLElement | null>(null);

onMounted(() => {
  // Aseguramos que no haya scroll mientras carga
  document.body.style.overflow = "hidden";

  // Timeline de GSAP para control exacto
  const tl = gsap.timeline({
    onComplete: () => {
      // Cuando termina la animación de salida:
      isVisible.value = false;
      document.body.style.overflow = "auto";
      window.dispatchEvent(new CustomEvent("app:ready"));
      document.documentElement.classList.add("app-ready");
    }
  });

  // 1. Animamos el logo (Aparece)
  tl.fromTo(
    logoRef.value,
    { opacity: 0, scale: 0.8, y: -20 },
    { opacity: 1, scale: 1, y: 0, duration: 0.6, ease: "back.out(1.7)" }
  );

  // 2. Animamos la pantalla blanca (Se desvanece)
  tl.to(
    loaderRef.value,
    { opacity: 0, duration: 0.5, ease: "power2.inOut" },
    "+=0.2"
  );
});
</script>

<style scoped>
.page-loader {
  position: fixed;
  inset: 0;
  z-index: 9999;
  display: grid;
  place-items: center;
  background-color: #ffffff;
  /* Previene parpadeos de FOUC */
  will-change: opacity;
}

.loading {
  width: clamp(266px, 22vw, 200px);
  height: auto;
  user-select: none;
  -webkit-user-drag: none;
  /* Inicia invisible para que GSAP tome el control sin parpadeos */
  opacity: 0;
  will-change: transform, opacity;
}
</style>

