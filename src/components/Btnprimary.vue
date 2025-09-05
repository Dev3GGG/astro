<template>
    <button class="find-out" href="">{{ text }}</button>
</template>

<script setup lang="ts">
interface Props {
    text: string
}

const props = defineProps<Props>()
const { text } = props
</script>

<style>
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
</style>
