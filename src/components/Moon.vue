<template>
    <primitive v-if="isReady" :object="model" />
</template>

<script setup lang="ts">
import { ref, onMounted, onBeforeMount, nextTick } from "vue"
import { gsap } from "gsap"
import { MotionPathPlugin } from "gsap/MotionPathPlugin"
import { useGLTF } from "@tresjs/cientos"
import type { Scene as ThreeScene } from "three"

gsap.registerPlugin(MotionPathPlugin);
const isReady = ref(false)
let model: ThreeScene | null = null
const { scene } = await useGLTF('/models/scene.gltf', { draco: true })

onBeforeMount(async () => {
    model = scene
})

onMounted(async () => {
    await nextTick()
    if (!model) return
    isReady.value = true
    const tl = gsap.timeline()
    tl.from(model.scale, {
        x: 0.7,
        y: 0.7,
        z: 0.8,
        duration: 2,
        ease: 'power2.out'
    }, 0)
    tl.to(model.rotation, {
        y: `+=${Math.PI * 2}`,
        duration: 30,
        ease: 'none',
        repeat: -1
    }, 0)

    tl.to(model.rotation, {
        x: `+=${Math.PI * 2}`,
        duration: 30,
        ease: 'none',
        repeat: -1
    }, 0)
    tl.to(model.position, {
        duration: 5,
        repeat: -1,
        ease: 'none',
        motionPath: {
            path: [
                { x: 0.02, y: 0 },
                { x: 0.00, y: 0 },
                { x: -0.10, y: 0 },
                { x: 0.00, y: 0 },
            ],
            curviness: 2,
            autoRotate: false
        },
        onUpdate() {
            if (!model) return
            const p = (this as unknown as gsap.core.Tween).progress()
            const z =
                p < 0.25 ? (p / 0.25) * 0.1 :
                    p < 0.50 ? 0.1 - ((p - 0.25) / 0.25) * 0.1 :
                        p < 0.75 ? -((p - 0.50) / 0.25) * 0.1 :
                            -0.1 + ((p - 0.75) / 0.25) * 0.1

            model.position.z = z
        }
    })
})
</script>
