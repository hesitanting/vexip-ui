<template>
  <label :class="className">
    <span :class="`${prefix}__signal`"></span>
    <span v-if="hasLabel || hasSlot" :class="[`${prefix}__label`, labelClass]">
      <slot>{{ label }}</slot>
    </span>
    <input
      type="checkbox"
      :class="`${prefix}__input`"
      :checked="currentChecked"
      :disabled="isDisabled"
      @change="handleChange()"
    />
  </label>
</template>

<script>
import formControl from '../../mixins/form-control'
import {
  isNull,
  findComponentUpward,
  removeArrayItem
} from '../../utils/common'

const { prefix } = require('../../style/basis/variable')

const groupName = 'CheckboxGroup'

export default {
  name: 'Checkbox',
  mixins: [formControl],
  model: {
    prop: 'checked',
    event: 'on-change'
  },
  props: {
    checked: {
      type: Boolean,
      default: false
    },
    label: {
      type: [String, Number],
      default: null
    },
    value: {
      type: [String, Number],
      default: null
    },
    labelClass: {
      type: [String, Array, Object],
      default: null
    },
    size: {
      default: 'default',
      validator(value) {
        return ['small', 'default', 'large'].includes(value)
      }
    },
    disabled: {
      type: Boolean,
      default: false
    },
    border: {
      type: Boolean,
      default: false
    },
    control: {
      type: Boolean,
      default: false
    },
    partial: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      prefix: `${prefix}-checkbox`,
      currentChecked: this.checked,
      currentPartial: this.partial,
      groupInstance: null,
      currentValue: isNull(this.value) ? this.label : this.value
    }
  },
  computed: {
    className() {
      const {
        prefix,
        currentChecked,
        disabled,
        size,
        border,
        control,
        currentPartial
      } = this

      return [
        prefix,
        {
          [`${prefix}--checked`]: currentChecked,
          [`${prefix}--disabled`]: disabled,
          [`${prefix}--${size}`]: size !== 'default',
          [`${prefix}--border`]: border,
          [`${prefix}--partial`]: control && currentPartial
        }
      ]
    },
    hasLabel() {
      return !isNull(this.label) && this.label !== ''
    },
    hasSlot() {
      return !!this.$slots.default
    },
    isDisabled() {
      if (this.groupInstance) {
        return this.groupInstance.disabled || this.disabled
      }

      return this.disabled
    }
  },
  watch: {
    checked(value) {
      this.handleChange(value, false)
    },
    currentChecked(value) {
      if (!this.control) {
        this.$nextTick(() => {
          if (this.groupInstance && this.hasLabel) {
            this.groupInstance.labels[this.label] = value
            this.groupInstance.updateValue()
          }
        })
      }
    },
    label(value, old) {
      if (!this.control && this.groupInstance && !isNull(value)) {
        if (!isNull(old)) {
          delete this.groupInstance.labels[old]
          delete this.groupInstance.valueMaps[old]
        }

        this.groupInstance.labels[value] = this.currentChecked
        this.groupInstance.valueMaps[value] = this.value
      }
    },
    value(value) {
      this.currentValue = isNull(value) ? this.label : value

      if (this.groupInstance) {
        this.groupInstance.valueMaps[this.label] = this.currentValue
      }
    },
    partial(value) {
      this.currentPartial = value
    }
  },
  created() {
    const groupInstance = findComponentUpward(this, groupName)

    if (groupInstance) {
      this.groupInstance = groupInstance

      if (this.control) {
        this.bindGroup(this.groupInstance)
      } else {
        groupInstance.items.push(this)

        if (this.hasLabel) {
          groupInstance.labels[this.label] = this.currentChecked
          this.groupInstance.valueMaps[this.label] = this.currentValue
        }
      }
    }
  },
  beforeDestroy() {
    if (this.groupInstance) {
      removeArrayItem(this.groupInstance.items, this)
    }
  },
  methods: {
    handleChange(checked = !this.currentChecked, emit = true) {
      if (this.control && this.groupInstance) {
        this.currentChecked = !!this.groupInstance.handleGlobelChange()
      } else {
        this.currentChecked = checked
      }

      if (emit) {
        this.$emit('on-change', checked)
      }
    },
    bindGroup(groupInstance) {
      if (groupInstance) {
        if (this.groupInstance) {
          this.groupInstance.$off('on-change', this.updatePartial)
        }

        this.groupInstance = groupInstance
        this.groupInstance.$on('on-change', this.updatePartial)
      }
    },
    unbindGroup() {
      if (this.groupInstance) {
        this.groupInstance.$off('on-change', this.updatePartial)
        this.groupInstance = null
      }
    },
    updatePartial(value) {
      if (this.control && this.groupInstance) {
        const checked = value.length === this.groupInstance.items.length

        this.currentChecked = checked
        this.currentPartial = value.length > 0 && !checked
      }
    }
  }
}
</script>
