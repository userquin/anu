<script lang="ts" setup>
import type { ExtractPropTypes } from 'vue'
import { typographyProps } from '@/components/typography/props'
import { typographySlots } from '@/components/typography/slots'
import { useLayer, useProps as useLayerProps } from '@/composables/useLayer'
import { configurable as configurableProp } from '@/composables/useProps'

const props = defineProps({
  ...typographyProps,

  ...useLayerProps({
    color: {
      default: 'primary',
    },
    variant: {
      default: 'light',
    },
  }),

  /**
   * prepend icon
   */
  icon: configurableProp,

  /**
   * append (close) icon
   */
  appendIcon: configurableProp,

  /**
   * Make alert dismissible using this prop. Adds close icon as appended icon.
   */
  dismissible: Boolean,

  /**
   * Hide/Show alert based on v-model value
   */
  modelValue: {
    type: Boolean,
    default: undefined,
  },
})

const emit = defineEmits<{

  /**
   * Emitted when append icon is clicked, including close icon in closable alert.
   */
  (e: 'click:appendIcon'): void

  /**
   * Emitted when `modelValue` is updated
   */
  (e: 'update:modelValue', value: (ExtractPropTypes<typeof props>)['modelValue']): void
}>()

defineOptions({
  name: 'AAlert',
})

// TODO: Move to slots file
const { default: _, ...alertTypographySlots } = typographySlots
const slots = {
  ...alertTypographySlots,

  /**
   * Default slot for rendering alert content
   */
  default: {},

  /**
   * Slot for appending custom content. Will override the close icon if `dismissible` is set to `true`.
   */
  append: {},
}
defineSlots<typeof slots>()

const isAlertVisible = useVModel(props, 'modelValue', emit, { defaultValue: true, passive: true })

const { getLayerClasses } = useLayer()
const { styles, classes } = getLayerClasses(
  toRef(props, 'color'),
  toRef(props, 'variant'),
  toRef(props, 'states'),
)

// 👉 Append icon
const appendIcon = props.appendIcon || (props.dismissible ? 'i-bx-x' : null)
function handleAppendIconClick() {
  isAlertVisible.value = false

  // Emit append icon click event
  emit('click:appendIcon')
}

const appendIconBindings = computed(() => {
  if (props.dismissible) {
    return {
      icon: appendIcon,
      ariaLabel: 'close',
    }
  }

  return {
    class: appendIcon,
  }
})

const _typographyProps = reactivePick(props, Object.keys(typographyProps) as Array<keyof typeof typographyProps>)
</script>

<template>
  <div
    role="alert"
    class="a-alert items-start w-full"
    :class="[
      ...classes,
      isAlertVisible ? 'flex' : 'hidden',
    ]"
    :style="styles"
  >
    <!-- ℹ️ We need div as wrapper with span having `vertical-align: text-top` to center the icon with the text -->
    <div v-if="props.icon">
      <i :class="props.icon" />
    </div>
    <div
      class="flex-grow"
      data-no-reference
    >
      <slot>
        <ATypography v-bind="_typographyProps">
          <!-- ℹ️ Recursively pass down slots to child -->
          <template
            v-for="name in Object.keys(alertTypographySlots)"
            #[name]="slotProps"
          >
            <slot
              :name="name"
              v-bind="slotProps || {}"
            />
          </template>
        </ATypography>
      </slot>
    </div>
    <div>
      <slot name="append">
        <Component
          :is="props.dismissible ? alertTypographySlots : 'i'"
          v-if="appendIcon"
          class="align-text-top"
          v-bind="appendIconBindings"
          @click="handleAppendIconClick"
        />
      </slot>
    </div>
  </div>
</template>
