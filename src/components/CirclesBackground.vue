<script setup lang="ts">
import { TresCanvas, useRenderLoop } from '@tresjs/core'
import { ref, onMounted, onBeforeUnmount } from 'vue'
import * as THREE from 'three'

type Particle = {
    pos: THREE.Vector3
    vel: THREE.Vector2
    mat: 0 | 1
    radius: number
}

const bounds = { x: 0.8, y: 0.35 }
const maxRadius = 0.08
const minRadius = 0.04
const count = 5
const restitution = 0.98

const textures = ref<THREE.Texture[]>([])
const materials = ref<THREE.MeshBasicMaterial[]>([])
const geometry = ref<THREE.SphereGeometry | null>(null)

const particles = ref<Particle[]>([])

function createRadialTexture(
    size = 512,
    inner = '#090979',
    mid = '#090979',
    outer = '#00D4FF',
    outerAlpha = 1
): THREE.Texture {
    const canvas = document.createElement('canvas')
    canvas.width = size
    canvas.height = size
    const ctx = canvas.getContext('2d')!

    const r = size / 2
    const g = ctx.createRadialGradient(r, r, 0, r, r, r)
    g.addColorStop(0.0, inner)
    g.addColorStop(0.35, mid)
    const outerColor = outerAlpha < 1 ? `rgba(${hexToRgb(outer)}, ${outerAlpha})` : outer
    g.addColorStop(1.0, outerColor)

    ctx.clearRect(0, 0, size, size)
    ctx.fillStyle = g
    ctx.fillRect(0, 0, size, size)

    const tex = new THREE.CanvasTexture(canvas)
    tex.colorSpace = THREE.SRGBColorSpace
    tex.minFilter = THREE.LinearFilter
    tex.magFilter = THREE.LinearFilter
    tex.generateMipmaps = false
    tex.needsUpdate = true
    return tex
}

function hexToRgb(hex: string) {
    const c = hex.replace('#', '')
    const bigint = parseInt(c, 16)
    const r = (bigint >> 16) & 255
    const g = (bigint >> 8) & 255
    const b = bigint & 255
    return `${r}, ${g}, ${b}`
}

function randomVel(): number {
    const v = THREE.MathUtils.randFloat(0.0002, 0.0006)
    return Math.random() > 0.5 ? v : -v
}

function spawnNonOverlapping(n: number): Particle[] {
    const arr: Particle[] = []
    for (let i = 0; i < n; i++) {
        const radius = THREE.MathUtils.lerp(minRadius, maxRadius, Math.random())
        let p: THREE.Vector3
        let attempts = 0
        do {
            p = new THREE.Vector3(
                THREE.MathUtils.randFloat(-bounds.x + radius, bounds.x - radius),
                THREE.MathUtils.randFloat(-bounds.y + radius, bounds.y - radius),
                0
            )
            attempts++
            if (attempts > 800) break
        } while (arr.some((q) => p.distanceTo(q.pos) < radius + q.radius))

        arr.push({
            pos: p,
            vel: new THREE.Vector2(randomVel(), randomVel()),
            mat: (Math.random() < 0.5 ? 0 : 1) as 0 | 1,
            radius
        })
    }
    return arr
}

onMounted(() => {
    const texBlue = createRadialTexture(512, '#1D4ED8', '#2563EB', '#60A5FA', 1)
    const texPurple = createRadialTexture(512, '#C7D2FF', '#A78BFA', '#592F94', 0.7)

    textures.value = [texBlue, texPurple]

    const matBlue = new THREE.MeshBasicMaterial({ map: texBlue })
    const matPurple = new THREE.MeshBasicMaterial({
        map: texPurple,
        transparent: true,
        depthWrite: false,
        blending: THREE.NormalBlending
    })
    materials.value = [matBlue, matPurple]

    geometry.value = new THREE.SphereGeometry(1, 32, 32)

    particles.value = spawnNonOverlapping(count)
})

onBeforeUnmount(() => {
    geometry.value?.dispose()
    materials.value.forEach((m) => m.dispose())
    textures.value.forEach((t) => t.dispose())
})

function handleWallBounce(p: Particle) {
    if (p.pos.x + p.radius > bounds.x) {
        p.pos.x = bounds.x - p.radius
        p.vel.x *= -1
    } else if (p.pos.x - p.radius < -bounds.x) {
        p.pos.x = -bounds.x + p.radius
        p.vel.x *= -1
    }
    if (p.pos.y + p.radius > bounds.y) {
        p.pos.y = bounds.y - p.radius
        p.vel.y *= -1
    } else if (p.pos.y - p.radius < -bounds.y) {
        p.pos.y = -bounds.y + p.radius
        p.vel.y *= -1
    }
}

function resolveParticleCollisions(ps: Particle[]) {
    for (let i = 0; i < ps.length; i++) {
        for (let j = i + 1; j < ps.length; j++) {
            const a = ps[i], b = ps[j]
            const dx = b.pos.x - a.pos.x
            const dy = b.pos.y - a.pos.y
            let dist = Math.hypot(dx, dy)

            if (dist === 0) {
                const angle = Math.random() * Math.PI * 2
                const eps = 1e-3
                a.pos.x -= Math.cos(angle) * eps
                a.pos.y -= Math.sin(angle) * eps
                dist = eps
            }

            const minDist = a.radius + b.radius
            if (dist < minDist) {
                const overlap = (minDist - dist) / 2
                const nx = dx / dist, ny = dy / dist
                a.pos.x -= nx * overlap; a.pos.y -= ny * overlap
                b.pos.x += nx * overlap; b.pos.y += ny * overlap
                const rvx = b.vel.x - a.vel.x
                const rvy = b.vel.y - a.vel.y
                const vn = rvx * nx + rvy * ny
                if (vn < 0) {
                    const jImp = -(1 + restitution) * vn / 2
                    a.vel.x -= jImp * nx; a.vel.y -= jImp * ny
                    b.vel.x += jImp * nx; b.vel.y += jImp * ny
                }
            }
        }
    }
}

const { onLoop } = useRenderLoop()
onLoop(() => {
    const ps = particles.value
    for (let i = 0; i < ps.length; i++) {
        const p = ps[i]
        p.pos.x += p.vel.x
        p.pos.y += p.vel.y
    }
    resolveParticleCollisions(ps)
    for (let i = 0; i < ps.length; i++) handleWallBounce(ps[i])
})
</script>

<template>
    <TresCanvas :dpr="[1, 2]" :alpha="true">
        <TresPerspectiveCamera :position="[0, 0, 1]" :look-at="[0, 0, 0]" />
        <TresAmbientLight :intensity="0.8" />
        <TresDirectionalLight :position="[1, 1, 1]" :intensity="1.2" />

        <TresMesh v-for="(p, i) in particles" :key="i" v-if="geometry && materials.length"
            :position="[p.pos.x, p.pos.y, p.pos.z]" :scale="[p.radius, p.radius, p.radius]" :geometry="geometry"
            :material="materials[p.mat]" />
    </TresCanvas>
</template>

<style>
canvas {
    display: block;
    width: 100%;
    height: 100%;
}
</style>