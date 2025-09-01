<template>
    <section class="tres-container" id="inicio">
        <div class="tres-card fade-up">
            <div class="grid fade-up">
                <div class="title-content fade-up">
                    <p class="title-part-one">TRANSFORMANDO IDEAS</p>
                    <h1 class="title-part-two">EN GRANDES<span><br />SOLUCIONES</span></h1>
                    <p>Lorem ipsum dolor sit amet consectetur, adipisicing elit. Reprehenderit, commodi? Ratione
                        necessitatibus, similique architecto sit ad incidunt rem voluptatum explicabo natus eaque fugiat
                        molestias illum veritatis, vel beatae autem nemo!</p>
                    <button class="find-out" aria-label="Descubre más sobre nuestras soluciones">Descúbrelo</button>
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
        </div>
    </section>
</template>
<script setup lang="ts">
import { onMounted } from 'vue'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import { TresCanvas } from '@tresjs/core'
import { BasicShadowMap, SRGBColorSpace, NoToneMapping } from 'three'
import Moon from './Moon.vue'

gsap.registerPlugin(ScrollTrigger)

const gl = {
    alpha: true,
    shadows: true,
    shadowMapType: BasicShadowMap,
    outputColorSpace: SRGBColorSpace,
    toneMapping: NoToneMapping,
}

onMounted(() => {
    gsap.utils.toArray<HTMLElement>('.fade-up').forEach(el => {
        gsap.from(el, {
            y: 30,
            opacity: 0,
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
})


</script>

<style>
.tres-container {
    position: relative;
    padding: 16px clamp(12px, 3vw, 32px) clamp(20px, 4vw, 40px);
    overflow: hidden;
}

.tres-card {
    position: relative;
    max-width: 1200px;
    margin: 0 auto;
    border-radius: 28px;
    outline: 1px solid #e9ecf5;
    padding: clamp(16px, 3vw, 28px) clamp(16px, 4vw, 36px);
    background: lch(47.9% 8.61 273.31 / 0.118);
}

.grid {
    display: grid;
    grid-template-columns: 1.1fr 1fr;
    align-items: center;
    gap: clamp(20px, 4vw, 48px);
    min-height: 600px;
}

@media (max-width: 1024px) {
    .grid {
        grid-template-columns: 1fr;
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
    font-weight: 100;
    font-size: clamp(14px, 2.1vw, 25px);
    color: black;
    opacity: 0.8;
    margin: 0 0 8px 0;
}

.title-part-two {
    color: #8aa2ff;
    margin: 0 0 clamp(10px, 1.6vw, 16px) 0;
    line-height: 0.95;
    font-weight: 800;
    font-size: clamp(36px, 6vw, 64px);
    letter-spacing: -0.02em;
}

.title-content p:not(.title-part-one) {
    margin: 0 0 clamp(12px, 2.5vw, 18px) 0;
    font-size: clamp(14px, 1.9vw, 16px);
    line-height: 1.5;
}

.find-out {
    appearance: none;
    border: 0;
    padding: clamp(10px, 1.6vw, 14px) clamp(20px, 3.2vw, 32px);
    border-radius: 999px;
    background: #4d78d3;
    color: #fff;
    font-weight: 700;
    letter-spacing: 0.02rem;
    cursor: pointer;
    font-size: clamp(14px, 1.8vw, 16px);
}

.find-out:hover {
    background: #3d63b4;
    box-shadow: 0 5px 5px rgba(61, 99, 180, 0.45);
}

.object {
    position: relative;
    min-height: clamp(260px, 43vw, 530px);
    border-radius: 24px;
    overflow: hidden;
}

@media (max-width: 600px) {
    .object {
        min-height: clamp(240px, 55vw, 420px);
    }
}

.object::before {
    position: absolute;
    inset: 0;
    background: rgba(107, 114, 128, 0.118);
    pointer-events: none;
    z-index: 1;
}

.object canvas {
    width: 100%;
    height: 100%;
    display: block;
    background: transparent;
}

.object canvas {
    position: relative;
    z-index: 0;
}
</style>
