<script setup lang="ts" generic="T extends any, O extends any">
import type { BlockState } from '~/types'

const DEV = false
const WIDTH = 10
const HEIGHT = 10
const BOOM_GENERATE_RATE = 0.4
const state = ref(
  Array.from(
    { length: HEIGHT },
    (_, y) => Array.from(
      { length: WIDTH },
      (_, x): BlockState => ({
        x,
        y,
        adjacentMines: 0,
        revealed: false,
      }),
    ),
  ),
)

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
  [0, 1],
]

const updateNumbers = () => {
  state.value.forEach((row, y) => {
    row.forEach((block, x) => {
      if (block.mine)
        return
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}

const generateMines = (initial: BlockState) => {
  for (const row of state.value) {
    for (const block of row) {
      // 第一次点击的时候 不应该点到炸弹
      if (Math.abs(initial.x - block.x) <= 1)
        continue
      if (Math.abs(initial.y - block.y) <= 1)
        continue
      block.mine = Math.random() < BOOM_GENERATE_RATE
    }
  }
  updateNumbers()
}

// 点击0的时候 需要递归翻开旁边所有0的格子
const expendZero = (block: BlockState) => {
  if (block.adjacentMines)
    return

  getSiblings(block).forEach((s) => {
    // 翻开所有的格子
    if (!s.revealed) {
      s.revealed = true
      expendZero(s)
    }
  })
}

// 刚进来是不生成的
let mineGenerated = false
const checkGameState = () => {
  if (!mineGenerated)
    return

  const blocks = state.value.flat()

  if (blocks.every(block => block.revealed || block.flagged)) {
    if (blocks.some(block => block.flagged && !block.mine))
      alert('You cheat!')
    else
      alert('You win!')
  }
}

watchEffect(checkGameState)

const onRightClick = (block: BlockState) => {
  if (block.revealed)
    return
  block.flagged = !block.flagged
}

const onClick = (block: BlockState) => {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  block.revealed = true
  // 1. 展开block
  if (block.mine)
    alert('BOOOOOOOM!')

  expendZero(block)
}

const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',
]

const getBlockClass = (block: BlockState) => {
  if (block.flagged)
    return 'bg-gray-500/10'

  if (!block.revealed)
    return 'bg-gray-500/10 hover:bg-gray/20'
  return block.mine ? 'bg-red/50' : numberColors[block.adjacentMines]
}

// 获取旁边8个格子的坐标
function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    // 判断越界情况
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
      return undefined
    return state.value[y2][x2]
  })
    .filter(Boolean) as BlockState[]
}
</script>

<template>
  <h1 text-xl mb-4>
    Minesweeper
  </h1>
  <div p-5>
    <div
      v-for="(row, y) in state"
      :key="y"
      flex="~"
      items-center
      justify-center
    >
      <button
        v-for="(block, x) in row"
        :key="x"
        w-10
        h-10
        border="1 gray-400/10"
        flex="~"
        items-center
        justify-center
        :class="getBlockClass(block)"
        @click="onClick(block)"
        @contextmenu.prevent="onRightClick(block)"
      >
        <template v-if="block.flagged">
          <div i-mdi-flag text-red />
        </template>
        <template v-else-if="block.revealed || DEV">
          <div v-if="block.mine" i-mdi-mine />
          <div v-else>
            {{ block.adjacentMines }}
          </div>
        </template>
      </button>
    </div>
  </div>
</template>
