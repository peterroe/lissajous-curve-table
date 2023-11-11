<script setup lang="ts">
import { onMounted, reactive, ref, watchEffect } from 'vue'
import gsap from 'gsap'

const colors = ['#ec6866', '#f2873d', '#f3c430', '#52d67a', '#609bf4', '#7a82f1', '#b67cf5']

let arr = new Array(7).fill(0).map(it => reactive({
  left: 0,
  top: 1,
}))

arr.forEach((it, i) => {
  gsap.timeline({
    defaults: {
      duration: 0.4 * (i + 1),
    }
  }).to(it, { left: 1, ease: 'sine.in' })
    .to(it, { left: 2, ease: 'sine.out' })
    .to(it, { left: 1, ease: 'sine.in' })
    .to(it, { left: 0, ease: 'sine.out' }).repeat(-1)


    gsap.timeline({
      defaults: {
        duration: 0.4 * (i + 1),
      }
    }).to(it, { top: 2, ease: 'sine.out' })
      .to(it, { top: 1, ease: 'sine.in' })
      .to(it, { top: 0, ease: 'sine.out' })
      .to(it, { top: 1, ease: 'sine.in' }).repeat(-1)
})

function blendColors(color1, color2) {
    // 将 HEX 转换为 RGB
    const rgb1 = parseInt(color1.slice(1), 16);
    const r1 = (rgb1 >> 16) & 255;
    const g1 = (rgb1 >> 8) & 255;
    const b1 = rgb1 & 255;

    const rgb2 = parseInt(color2.slice(1), 16);
    const r2 = (rgb2 >> 16) & 255;
    const g2 = (rgb2 >> 8) & 255;
    const b2 = rgb2 & 255;

    // 计算平均值
    const blendedR = Math.round((r1 + r2) / 2);
    const blendedG = Math.round((g1 + g2) / 2);
    const blendedB = Math.round((b1 + b2) / 2);

    // 将 RGB 转换回 HEX
    const blendedColor = "#" + ((1 << 24) + (blendedR << 16) + (blendedG << 8) + blendedB).toString(16).slice(1);

    return blendedColor;
}

const shuffleI = new Array(7).fill(0).map((_, i) => i)
const shuffleJ = new Array(7).fill(0).map((_, i) => i)

const draw = (ctx: CanvasRenderingContext2D) => {
    shuffleI.forEach((iOrder, i) => {
      shuffleJ.forEach((jOrder, j) => {
        const left = Math.round(arr[iOrder].left * 40)
        const top = Math.round(arr[jOrder].top * 40)
        ctx.beginPath();
        ctx.arc(left + 110 * i + 30, top + 110 * j + 30, 1.5, 0, 2 * Math.PI)
        ctx.fillStyle = blendColors(colors[i], colors[j]);
        ctx.fill(); // 填充颜色
      })
    })
    requestAnimationFrame(() => draw(ctx))
}

const canvas = ref<HTMLCanvasElement>()

onMounted(() => {
  if(!canvas.value) throw new Error('canvas is not ready')
  const ctx = canvas.value.getContext('2d')!
  draw(ctx)
})

const replay = () => {
  if(!canvas.value) throw new Error('canvas is not ready')
  const ctx = canvas.value.getContext('2d')!
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)
  shuffleI.sort()
  shuffleJ.sort()
  draw(ctx)
}

const random = () => {
  if(!canvas.value) throw new Error('canvas is not ready')
  const ctx = canvas.value.getContext('2d')!
  ctx.clearRect(0, 0, canvas.value.width, canvas.value.height)
  shuffleI.sort(() => Math.random() - 0.5)
  shuffleJ.sort(() => Math.random() - 0.5)
  draw(ctx)
}

const jumpGitHub = () => {
  location.href = 'https://github.com/peterroe/lissajous-curve-table'
}

</script>

<template>
  <div class="main" bg="[#161618]" h-screen text-white text-center flex justify="center" items-center>
    <div class="body" w-200 mx-auto position-absolute >
      <div class="header" text-5xl>Lissajous curse table</div>
      <div my-6 flex justify="center">
        <div w-20 px-3 py-2 gap-9 text-lg rounded-8 border-none cursor="pointer" bg-red mr-3 @click="replay">replay</div>
        <div w-20 px-3 py-2 gap-9 text-lg rounded-8 border-none cursor="pointer" bg-red mr-3 @click="random">random</div>
          <div justify="center" items-center rounded-full flex cursor="pointer" bg="white" w-11 h-11 @click="jumpGitHub">
            <svg w-8 h-8 role="img" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg"><path d="M12 .297c-6.63 0-12 5.373-12 12 0 5.303 3.438 9.8 8.205 11.385.6.113.82-.258.82-.577 0-.285-.01-1.04-.015-2.04-3.338.724-4.042-1.61-4.042-1.61C4.422 18.07 3.633 17.7 3.633 17.7c-1.087-.744.084-.729.084-.729 1.205.084 1.838 1.236 1.838 1.236 1.07 1.835 2.809 1.305 3.495.998.108-.776.417-1.305.76-1.605-2.665-.3-5.466-1.332-5.466-5.93 0-1.31.465-2.38 1.235-3.22-.135-.303-.54-1.523.105-3.176 0 0 1.005-.322 3.3 1.23.96-.267 1.98-.399 3-.405 1.02.006 2.04.138 3 .405 2.28-1.552 3.285-1.23 3.285-1.23.645 1.653.24 2.873.12 3.176.765.84 1.23 1.91 1.23 3.22 0 4.61-2.805 5.625-5.475 5.92.42.36.81 1.096.81 2.22 0 1.606-.015 2.896-.015 3.286 0 .315.21.69.825.57C20.565 22.092 24 17.592 24 12.297c0-6.627-5.373-12-12-12"></path></svg>
          </div>
      </div>
      <canvas id="canvas" ref="canvas" width="800" height="800"  ></canvas>
    </div>
  </div>
</template>

<style scoped>
.main {
  text-align: center;
}

@media screen and (max-width: 800px) {
    canvas, .body {
        width: calc(100% - 20px); /* 设置宽度为100% */
        height: auto; /* 高度自动调整以保持宽高比 */
        aspect-ratio: 1;
    }
    .body {
      width: 100%;
    }
}
</style>
