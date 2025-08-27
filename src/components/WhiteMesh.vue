<script setup lang="ts">
import { OBJLoader } from 'three/examples/jsm/loaders/OBJLoader.js'
import { gsap } from 'gsap'
import { MotionPathPlugin } from "gsap/MotionPathPlugin";
import { ref, onMounted } from 'vue';
import type { Group, Mesh } from 'three'

gsap.registerPlugin(MotionPathPlugin);
const object_3D: Group = await new OBJLoader().loadAsync('/models/white_mesh.obj')
const obj_Ref = ref<Mesh | null>(null)
onMounted(() => {
    if (obj_Ref.value) {
        gsap.to(obj_Ref.value.rotation, {
            y: `+=${Math.PI * 2}`,
            duration: 15,
            ease: 'none',
            repeat: -1,
        })

        gsap.to(obj_Ref.value.position, {
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
                if (!obj_Ref.value) return
                const z =
                    current_progress < 0.25 ? (current_progress / 0.25) * 0.1 :
                        current_progress < 0.50 ? 0.1 - ((current_progress - 0.25) / 0.25) * 0.1 :
                            current_progress < 0.75 ? -((current_progress - 0.50) / 0.25) * 0.1 :
                                -0.1 + ((current_progress - 0.75) / 0.25) * 0.1
                obj_Ref.value.position.z = z
            }
        })
    }
})
</script>
<template>
    <primitive :object="object_3D" ref="obj_Ref" />
</template>