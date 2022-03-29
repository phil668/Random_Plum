<template>
  <div>
    <canvas
      :width="width"
      :height="height"
      :style="{ border: '1px solid #fff' }"
      ref="el"
    ></canvas>
    <img :src="flower" alt="" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue'
import flower from '../assets/flower.svg'
const el = ref<HTMLCanvasElement>()
const width = ref<number>(600)
const height = ref<number>(600)
const pendingTask: Function[] = []
const frameCount = ref<number>(3)
const image = ref<HTMLImageElement>()
const ctx = computed(() => {
  return el.value?.getContext('2d')!
})

interface Point {
  x: number
  y: number
}

interface Branch {
  start: Point
  length: number
  theta: number
}
function init() {
  image.value = new Image()
  image.value.src = flower
  ctx.value.strokeStyle = '#dee1e6'
  ctx.value.beginPath()
}

function lineTo(p1: Point, p2: Point) {
  ctx.value.moveTo(p1.x, p1.y)
  ctx.value.lineTo(p2.x, p2.y)
  ctx.value.stroke()
}

// 获得结束端点的坐标
function getEndPoint(branch: Branch): Point {
  const { start, length, theta } = branch
  return {
    x: start.x + length * Math.cos(theta),
    y: start.y + length * Math.sin(theta),
  }
}

function drawLine(branch: Branch) {
  const { start } = branch
  const endPoint = getEndPoint(branch)
  lineTo(start, endPoint)
  // 随机生成花瓣
  if (Math.random() < 0.5) {
    ctx.value.drawImage(
      image.value as HTMLImageElement,
      endPoint.x - 5,
      endPoint.y - 5,
      10,
      10
    )
  }
}

function step(branch: Branch, depth: number = 0): void {
  drawLine(branch)
  const end = getEndPoint(branch)
  if (depth < 4 || Math.random() < 0.5) {
    pendingTask.push(() =>
      step(
        {
          start: end,
          length: branch.length + (Math.random() * 10 - 5),
          theta: branch.theta - 0.3 * Math.random(),
        },
        depth + 1
      )
    )
  }
  if (depth < 4 || Math.random() < 0.5) {
    pendingTask.push(() => {
      step(
        {
          start: end,
          length: branch.length + (Math.random() * 10 - 5),
          theta: branch.theta + 0.3 * Math.random(),
        },
        depth + 1
      )
    })
  }
}

function frame() {
  const tasks = [...pendingTask]
  pendingTask.length = 0
  tasks.forEach((fn) => fn())
}

function startFrame() {
  requestAnimationFrame(() => {
    frameCount.value += 1
    // 10帧画一次
    if (frameCount.value % 3 === 0) {
      frame()
    }
    startFrame()
  })
}

onMounted(() => {
  init()
  const branch: Branch = {
    start: { x: width.value / 2, y: height.value },
    length: 50,
    theta: -Math.PI / 2,
  }
  step(branch)
  startFrame()
  console.log(flower)
})
</script>

<style></style>
