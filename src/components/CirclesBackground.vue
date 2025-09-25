<script setup lang="ts">
import { TresCanvas, useRenderLoop } from '@tresjs/core'
import { ref, onMounted, onBeforeUnmount, reactive } from 'vue'
import * as THREE from 'three'

type Particle = {
    pos: THREE.Vector3
    vel: THREE.Vector2
    mat: 0 | 1
    radius: number
}

const bounds = reactive({ x: 0.8, y: 0.4 })
const maxRadius = 0.08
const minRadius = 0.04
const count = 5
const restitution = 0.98

const textures = ref<THREE.Texture[]>([])
const materials = ref<THREE.MeshBasicMaterial[]>([])
const geometry = ref<THREE.CircleGeometry | null>(null)

const particles = ref<Particle[]>([])

function update_bounds() {
    if (window.innerWidth <= 600) {
        bounds.x = 0.26
        bounds.y = 0.42
    } else {
        bounds.x = 0.9
        bounds.y = 0.4
    }
}

function create_linear_texture(
    size = 512,
    startColor = '#090979',
    midColor = '#090979',
    endColor = '#00D4FF',
    endAlpha = 1
): THREE.Texture {
    const canvas = document.createElement('canvas')
    canvas.width = size
    canvas.height = size
    const ctx = canvas.getContext('2d')!

    const g = ctx.createLinearGradient(0, 0, size, size)
    
    g.addColorStop(0.0, startColor)
    g.addColorStop(0.5, midColor)

    const finalEndColor = endAlpha < 1 ? `rgba(${hex_to_rgb(endColor)}, ${endAlpha})` : endColor
    g.addColorStop(1.0, finalEndColor)

    ctx.clearRect(0, 0, size, size)
    ctx.fillStyle = g
    ctx.fillRect(0, 0, size, size)

    const texture = new THREE.CanvasTexture(canvas)
    texture.colorSpace = THREE.SRGBColorSpace
    texture.minFilter = THREE.LinearFilter
    texture.magFilter = THREE.LinearFilter
    texture.generateMipmaps = false
    texture.needsUpdate = true
    return texture
}

function hex_to_rgb(hex: string) {
    const c = hex.replace('#', '')
    const bigint = parseInt(c, 16)
    const r = (bigint >> 16) & 255
    const g = (bigint >> 8) & 255
    const b = bigint & 255
    return `${r}, ${g}, ${b}`
}

function movement_speed(): number {
    const v = THREE.MathUtils.randFloat(0.0002, 0.0006)
    return Math.random() > 0.5 ? v : -v
}

function create_particles(n: number): Particle[] {
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
            vel: new THREE.Vector2(movement_speed(), movement_speed()),
            mat: (Math.random() < 0.5 ? 0 : 1) as 0 | 1,
            radius
        })
    }
    return arr
}

onMounted(() => {
    update_bounds()
    window.addEventListener('resize', update_bounds)
    const texBlue = create_linear_texture(512, '#60A5FA', '#60A5FA', '#1D4ED7', 1)
    const texPurple = create_linear_texture(512, '#592F94', '#A78BFA', '#C7D2FF', 1)

    textures.value = [texBlue, texPurple]

    const matBlue = new THREE.MeshBasicMaterial({ map: texBlue })
    const matPurple = new THREE.MeshBasicMaterial({
        map: texPurple,
        transparent: true,
        depthWrite: false,
        blending: THREE.NormalBlending
    })
    materials.value = [matBlue, matPurple]

    geometry.value = new THREE.CircleGeometry(1, 32, 32)

    particles.value = create_particles(count)
})

onBeforeUnmount(() => {
    window.removeEventListener('resize', update_bounds)
    geometry.value?.dispose()
    materials.value.forEach((m) => m.dispose())
    textures.value.forEach((t) => t.dispose())
})

function particle_rebound(particle: Particle) {
    if (particle.pos.x + particle.radius > bounds.x) {
        particle.pos.x = bounds.x - particle.radius
        particle.vel.x *= -1
    } else if (particle.pos.x - particle.radius < -bounds.x) {
        particle.pos.x = -bounds.x + particle.radius
        particle.vel.x *= -1
    }
    if (particle.pos.y + particle.radius > bounds.y) {
        particle.pos.y = bounds.y - particle.radius
        particle.vel.y *= -1
    } else if (particle.pos.y - particle.radius < -bounds.y) {
        particle.pos.y = -bounds.y + particle.radius
        particle.vel.y *= -1
    }
}

function particle_collision(particle: Particle[]) {
    for (let i = 0; i < particle.length; i++) {
        for (let j = i + 1; j < particle.length; j++) {
            const a = particle[i], b = particle[j]
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
    const particle = particles.value
    for (let i = 0; i < particle.length; i++) {
        const p = particle[i]
        p.pos.x += p.vel.x
        p.pos.y += p.vel.y
    }
    particle_collision(particle)
    for (let i = 0; i < particle.length; i++) particle_rebound(particle[i])
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
