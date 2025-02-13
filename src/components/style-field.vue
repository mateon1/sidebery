<template lang="pug">
.StyleField(:data-active="!!props.value")
  .label
    .desc {{translate(props.label) || props.name}}
    .var(v-if="props.name") {{props.name}}
  .input-group
    .color-wrapper(v-if="isColor")
      input.color(
        type="color"
        :style="{ opacity: colorOpacity }"
        :value="colorValue"
        @input="onColorInput")
    TextInput.text-input(
      :value="props.value"
      :line="true"
      :or="props.or"
      @update:value="onInput"
      @change="onChange"
      @keydown="onKeyDown")
    ToggleInput.toggle(:value="props.active", @update:value="toggle")
</template>

<script lang="ts" setup>
import { ref, computed } from 'vue'
import Utils from 'src/utils'
import { translate } from 'src/dict'
import TextInput from './text-input.vue'
import ToggleInput from './toggle-input.vue'

interface StyleFieldProps {
  value: string
  active: boolean
  or: string
  label: string
  name?: string
  isColor: boolean
}

const emit = defineEmits(['update:value', 'change', 'toggle'])
const props = defineProps<StyleFieldProps>()

const opaq = ref('ff')

const colorValue = computed((): string => {
  if (!props.value) return '#000000'

  if (props.value[0] === '#' && props.value.length === 7) return props.value.toLowerCase()

  const rgb = Utils.toRGBA(props.value)
  if (!rgb) return '#000000'

  return `#${rgb[0].toString(16)}${rgb[1].toString(16)}${rgb[2].toString(16)}`
})
const colorOpacity = computed((): number => {
  const num = parseInt(opaq.value, 16)
  if (isNaN(num)) return 1
  return num / 255
})

function onColorInput(e: Event): void {
  const len = (e.target as HTMLInputElement).value.length
  if (len < 7) (e.target as HTMLInputElement).value += '0'.repeat(6 - len)
  emit('update:value', (e.target as HTMLInputElement).value + opaq.value)
}

function onInput(val: string): void {
  emit('update:value', val)
}

function onKeyDown(e: KeyboardEvent): void {
  if (e.key === 'Escape' && props.active) toggle()
  if (e.key === 'Enter') toggle()
}

function onChange(): void {
  emit('change', props.value)
}

function toggle(): void {
  emit('toggle')
}
</script>
