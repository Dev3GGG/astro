<template>
  <div class="bg-3d" aria-hidden="true">
    <CirclesBackground client:only />
  </div>
  <section class="tres-container fade-up" id="home">
    <div class="grid fade-up">
      <div class="title-content fade-up">
        <p class="title-part-one">TRANSFORMANDO IDEAS</p>
        <h1 class="title-part-two">EN GRANDES<span><br />SOLUCIONES</span></h1>
        <p>
          Lorem ipsum dolor sit amet consectetur, adipisicing elit. Reprehenderit, commodi? Ratione
          necessitatibus, similique architecto sit ad incidunt rem voluptatum explicabo natus eaque fugiat
          molestias illum veritatis, vel beatae autem nemo!
        </p>
        <ButtonMain text="Descúbrelo" />
      </div>

      <div class="object fade-up">
        <TresCanvas v-bind="gl">
          <TresPerspectiveCamera :position="[0, 0, 3]" />
          <TresAmbientLight :intensity="0.9" />
          <TresDirectionalLight :position="[5, 5, 5]" :intensity="2" />
          <Suspense>
            <Moon />
          </Suspense>
        </TresCanvas>
      </div>
    </div>
  </section>
</template>

<script setup lang="ts">
import { onBeforeUnmount, onMounted } from 'vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { TresCanvas } from '@tresjs/core'
import { BasicShadowMap, SRGBColorSpace, NoToneMapping } from 'three'
import Moon from '@components/Moon.vue'
import ButtonMain from '@components/Btnprimary.vue'
import CirclesBackground from './CirclesBackground.vue'

gsap.registerPlugin(ScrollTrigger)

const gl = {
  alpha: true,
  shadows: true,
  shadowMapType: BasicShadowMap,
  outputColorSpace: SRGBColorSpace,
  toneMapping: NoToneMapping,
}

function runFadeUp() {
  gsap.utils.toArray<HTMLElement>('.fade-up').forEach((el) => {
    gsap.from(el, {
      y: 30,
      autoAlpha: 0,
      duration: 0.7,
      ease: 'power2.out',
      scrollTrigger: {
        trigger: el,
        start: 'top 85%',
        toggleActions: 'play none none none',
        once: true,
      },
    })
  })
  ScrollTrigger.refresh()
}

function onTresReady() {
  runFadeUp()
}

onMounted(() => {
  if (document.documentElement.classList.contains('tres-ready')) {
    runFadeUp()
  } else {
    window.addEventListener('tres:ready', onTresReady, { once: true })
  }
})

onBeforeUnmount(() => {
  window.removeEventListener('tres:ready', onTresReady)
})
</script>

<style>
.tres-container {
  position: relative;
  isolation: isolate;
  max-width: 1200px;
  border-radius: 28px;
  margin-bottom: 60px;
  outline: 1px solid #e9ecf5;
  padding: clamp(20px, 3vw, 28px) clamp(20px, 4vw, 36px);
  background: lch(47.9% 8.61 273.31 / 0.118);
  overflow: hidden;
  padding-block-end: clamp(20px, 4vw, 40px);
  padding-block-start: 20px;
  padding-inline: clamp(12px, 3vw, 32px);
}

.bg-3d {
  position: absolute;
  top: -40px;
  left: 0;
  right: 0;
  width: 100%;
  min-height: clamp(650px, 65vw, 960px);
  z-index: 0;
  pointer-events: none !important;
}

.grid {
  position: relative;
  z-index: 9999;
  display: grid;
  grid-template-columns: 1.1fr 1fr;
  align-items: center;
  gap: clamp(20px, 4vw, 48px);
  min-height: 550px;
}

@media (max-width: 1024px) {
  .tres-container {
    margin: 40px 20px 40px 20px;
  }

  .grid {
    grid-template-columns: 1fr;
    min-height: auto;
  }

  .title-content {
    order: 1;
  }

  .object {
    order: 2;
  }
}

.title-part-one {
  letter-spacing: 0.18em;
  text-transform: uppercase;
  font-size: clamp(18px, 2.1vw, 29px);
  color: rgba(0, 0, 0, 1);
  opacity: 0.8;
  margin: 0 0 8px 0;
}

.title-part-two {
  color: rgba(194, 205, 255, 1);
  margin: 0 0 clamp(10px, 1.6vw, 16px) 0;
  line-height: 0.95;
  font-weight: 900;
  font-size: clamp(36px, 6vw, 64px);
  letter-spacing: -0.02em;
}

.title-content p:not(.title-part-one) {
  margin: 0 0 clamp(12px, 2.5vw, 18px) 0;
  font-size: clamp(14px, 1.9vw, 16px);
  line-height: 1.5;
}

.object {
  position: relative;
  min-height: clamp(260px, 43vw, 530px);
  border-radius: 24px;
  overflow: hidden;
}

@media (max-width: 600px) {
  .bg-3d {
    top: 60px;
  }

  .object {
    min-height: clamp(240px, 55vw, 420px);
  }
}

.object canvas {
  width: 100%;
  height: 100%;
  display: block;
  background: transparent;
  position: relative;
  z-index: 0;
  pointer-events: none !important;
}
</style>
