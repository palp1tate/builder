<!--
 * @Author: Zhang Zhi Yang
 * @Date: 2024-01-25 16:13:37
 * @LastEditors: Zhang Zhi Yang
 * @LastEditTime: 2024-03-13 14:32:19
 * @FilePath: \spx-gui\src\components\stage-viewer\SpriteLayer.vue
 * @Description: 
-->
<template>
  <v-layer
    ref="layer"
    :config="{
      name: 'sprite',
      x: props.offsetConfig.offsetX,
      y: props.offsetConfig.offsetY
    }"
  >
    <template v-for="sprite in sortedSprites" :key="sprite.name">
      <!-- v-if="sprite.config.visible" -->
      <SpriteComp
        :sprite="sprite"
        :map-size="props.mapSize"
        :selected="selectedSpriteNames.includes(sprite.name)"
        @on-drag-move="onSpriteDragMove"
        @on-sprite-apperance-change="onSpriteApperanceChange"
      >
      </SpriteComp>
    </template>
  </v-layer>
</template>
<script setup lang="ts">
// ----------Import required packages / components-----------
import SpriteComp from './Sprite.vue'
import { computed } from 'vue'
import type { SpriteDragMoveEvent, SpriteApperanceChangeEvent } from './common'
import type { Sprite } from '@/models/sprite'
import type { Size } from '@/models/common'

const props = defineProps<{
  offsetConfig: { offsetX: number; offsetY: number }
  mapSize: Size
  spriteList: Sprite[]
  zorder: Array<string | Object>
  selectedSpriteNames: string[]
}>()
const emits = defineEmits<{
  (e: 'onSpriteDragMove', event: SpriteDragMoveEvent): void
  (e: 'onSpriteApperanceChange', event: SpriteApperanceChangeEvent): void
}>()

// spritelist sort by zorder config
const sortedSprites = computed(() => {
  const spriteMap = new Map<string, Sprite>()
  props.spriteList.forEach((sprite) => {
    spriteMap.set(sprite.name, sprite)
  })
  const list: Sprite[] = []
  // temporarily only sprite item  can be rendered on stage
  props.zorder.forEach((item) => {
    if (typeof item === 'string' && spriteMap.has(item)) {
      list.push(spriteMap.get(item) as Sprite)
    }
  })
  return list
})

const onSpriteDragMove = (e: SpriteDragMoveEvent) => {
  emits('onSpriteDragMove', e)
}

const onSpriteApperanceChange = (e: SpriteApperanceChangeEvent): void => {
  emits('onSpriteApperanceChange', e)
}
</script>
