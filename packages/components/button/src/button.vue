<template>
  <button
    ref="buttonRef"
    :class="[
      'el-button',
      buttonType ? 'el-button--' + buttonType : '',
      buttonSize ? 'el-button--' + buttonSize : '',
      {
        'is-disabled': buttonDisabled,
        'is-loading': loading,
        'is-plain': plain,
        'is-round': round,
        'is-circle': circle,
      },
    ]"
    :disabled="buttonDisabled || loading"
    :autofocus="autofocus"
    :type="nativeType"
    :style="buttonStyle"
    @click="handleClick"
  >
    <el-icon v-if="loading" class="is-loading">
      <loading />
    </el-icon>
    <el-icon v-else-if="icon">
      <component :is="icon" />
    </el-icon>
    <span
      v-if="$slots.default"
      :class="{ 'el-button__text--expand': shouldAddSpace }"
    >
      <slot></slot>
    </span>
  </button>
</template>

<script lang="ts">
import { computed, inject, defineComponent, Text, ref } from 'vue'
import { useCssVar } from '@vueuse/core'
import { ElIcon } from '@element-plus/components/icon'
import { useFormItem, useGlobalConfig } from '@element-plus/hooks'
import { buttonGroupContextKey } from '@element-plus/tokens'
import { Loading } from '@element-plus/icons'

import { lighten, darken } from '@element-plus/utils/color'

import { buttonEmits, buttonProps } from './button'

export default defineComponent({
  name: 'ElButton',

  components: {
    ElIcon,
    Loading,
  },

  props: buttonProps,
  emits: buttonEmits,

  setup(props, { emit, slots }) {
    const buttonRef = ref()
    const buttonGroupContext = inject(buttonGroupContextKey, undefined)
    const globalConfig = useGlobalConfig()
    const autoInsertSpace = computed(() => {
      return props.autoInsertSpace ?? globalConfig?.button.autoInsertSpace
    })

    // add space between two characters in Chinese
    const shouldAddSpace = computed(() => {
      const defaultSlot = slots.default?.()
      if (autoInsertSpace.value && defaultSlot?.length === 1) {
        const slot = defaultSlot[0]
        if (slot?.type === Text) {
          const text = slot.children
          return /^\p{Unified_Ideograph}{2}$/u.test(text as string)
        }
      }
      return false
    })

    const {
      form,
      size: buttonSize,
      disabled: buttonDisabled,
    } = useFormItem({
      size: computed(() => buttonGroupContext?.size),
    })
    const buttonType = computed(
      () => props.type || buttonGroupContext?.type || 'default'
    )

    // calculate hover & active color by color
    const typeColor = useCssVar(`--el-color-${props.type}`)
    const buttonStyle = computed(() => {
      let styles = {}

      const buttonColor = props.color || typeColor.value

      if (buttonColor) {
        const darkenBgColor = darken(buttonColor, 0.1)
        if (props.plain) {
          styles = {
            '--el-button-bg-color': lighten(buttonColor, 0.9),
            '--el-button-text-color': buttonColor,
            '--el-button-hover-text-color': 'var(--el-color-white)',
            '--el-button-hover-bg-color': buttonColor,
            '--el-button-hover-border-color': buttonColor,
            '--el-button-active-bg-color': darkenBgColor,
            '--el-button-active-text-color': 'var(--el-color-white)',
            '--el-button-active-border-color': darkenBgColor,
          }
        } else {
          const lightenBgColor = lighten(buttonColor)
          styles = {
            '--el-button-bg-color': buttonColor,
            '--el-button-border-color': buttonColor,
            '--el-button-hover-bg-color': lightenBgColor,
            '--el-button-hover-border-color': lightenBgColor,
            '--el-button-active-bg-color': darkenBgColor,
            '--el-button-active-border-color': darkenBgColor,
          }
        }

        if (buttonDisabled.value) {
          const disabledButtonColor = lighten(buttonColor, 0.5)
          styles['--el-button-disabled-bg-color'] = disabledButtonColor
          styles['--el-button-disabled-border-color'] = disabledButtonColor
        }
      }

      return styles
    })

    const handleClick = (evt: MouseEvent) => {
      if (props.nativeType === 'reset') {
        form?.resetFields()
      }
      emit('click', evt)
    }

    return {
      buttonRef,
      buttonStyle,

      buttonSize,
      buttonType,
      buttonDisabled,

      shouldAddSpace,

      handleClick,
    }
  },
})
</script>
