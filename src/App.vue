<script setup lang="ts">
const el = $ref<HTMLCanvasElement>()
const ctx = $computed(() => el!.getContext('2d')!)

const WIDTH = 600
const HEIGHT = 600

interface Branch {
  start: Point
  length: number
  angle: number
}

interface Point {
  x: number
  y: number
}

const pendingTasks: () => void = []

function init() {
  ctx.strokestyle = '#000'
  step({
    start: { x: WIDTH / 2, y: HEIGHT },
    length: 20,
    angle: -Math.PI / 2,
  })
}

function step(b: Branch, depth = 0) {
  const endPoint = getEndPoint(b)
  drawBranch(b)
  if (depth < 2 || Math.random() < 0.5) {
    pendingTasks.push(() => {
      step({
        start: endPoint,
        length: b.length + (Math.random() * 10 - 5),
        angle: b.angle - 0.3 * Math.random(),
      }, depth + 1)
    })
  }
  if (depth < 2 || Math.random() < 0.5) {
    pendingTasks.push(() => {
      step({
        start: endPoint,
        length: b.length + (Math.random() * 10 - 5),
        angle: b.angle + 0.3 * Math.random(),
      }, depth + 1)
    })
  }
}

function frame() {
  const tasks = [...pendingTasks]
  pendingTasks.length = 0
  tasks.forEach(fn => fn())
}

let frameCount = 0
function startFrame() {
  requestAnimationFrame(() => {
    frameCount += 1
    if (frameCount % 3) {
      frame()
    }
    startFrame()
  })
}

startFrame()

function getEndPoint(b: branch) {
  return {
    x: b.start.x + b.length * Math.cos(b.angle),
    y: b.start.y + b.length * Math.sin(b.angle),
  }
}

function lineTo(p1: Point, p2: Point) {
  ctx.beginPath() // Start a new path
  ctx.moveTo(p1.x, p1.y) // Move the pen to (30, 50)
  ctx.lineTo(p2.x, p2.y) // Draw a line to (150, 100)
  ctx.stroke() // Render the path
}

function drawBranch(b: Branch) {
  const end = getEndPoint(b)
  lineTo(b.start, end)
}

onMounted(() => {
  init()
})
</script>

<template>
  <canvas ref="el" width="600" height="600" border />
</template>

<style scoped>
.border {
  color: black;
}
</style>
