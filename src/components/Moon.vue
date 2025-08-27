<template>
    <primitive v-if="isReady" :object="model" />
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue"
import { gsap } from "gsap"
import { MotionPathPlugin } from "gsap/MotionPathPlugin"
import { useGLTF } from "@tresjs/cientos"

gsap.registerPlugin(MotionPathPlugin);
const isReady = ref(false)

const { scene } = await useGLTF('/models/scene.gltf', { draco: true })
const model = scene

onMounted(() => {
    setTimeout(() => {
        isReady.value = true
        gsap.from(model.scale, { x: 0.7, y: 0.7, z: 0.8, duration: 3, ease: 'zoom' })

        gsap.to(model.rotation, {
            y: `+=${Math.PI * 2}`,
            duration: 30,
            ease: 'none',
            repeat: -1,
        })
        gsap.to(model.rotation, {
            x: `+=${Math.PI * 2}`,
            duration: 30,
            ease: 'none',
            repeat: -1,
        })

        gsap.to(model.position, {
            duration: 5,
            repeat: -1,
            ease: 'none',
            motionPath: {
                path: [
                    { x: 0.02, y: 0 },
                    { x: 0, y: 0 },
                    { x: -0.1, y: 0 },
                    { x: 0, y: 0 },
                ],
                curviness: 2,
                autoRotate: true,
            },
            onUpdate() {
                const current_progress = this.progress()
                if (!model) return
                const z =
                    current_progress < 0.25 ? (current_progress / 0.25) * 0.1 :
                        current_progress < 0.50 ? 0.1 - ((current_progress - 0.25) / 0.25) * 0.1 :
                            current_progress < 0.75 ? -((current_progress - 0.50) / 0.25) * 0.1 :
                                -0.1 + ((current_progress - 0.75) / 0.25) * 0.1
                model.position.z = z
            }
        })
    }, 1200)
})
</script>
