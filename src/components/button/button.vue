<template>
  <button
    :class="className"
    :style="style"
    :type="buttonType"
    :disabled="disabled"
    @click.left="handleClick"
    @animationend="test"
  >
    <div v-if="loading" :class="`${prefixCls}__icon`">
      <slot name="loading">
        <Icon pulse :name="loadingIcon"></Icon>
      </slot>
    </div>
    <div v-if="icon && !loading" :class="`${prefixCls}__icon`">
      <Icon :name="icon"></Icon>
    </div>
    <slot></slot>
  </button>
</template>

<script>
import Icon from '../icon'
import 'vue-awesome/icons/spinner'

const { prefix } = require('../../style/basis/variable')

export default {
  name: 'Button',
  components: {
    Icon
  },
  props: {
    type: {
      validator(value) {
        return [
          'default',
          'primary',
          'dashed',
          'text',
          'info',
          'success',
          'warning',
          'error'
        ].includes(value)
      },
      default: 'default'
    },
    simple: {
      type: Boolean,
      default: false
    },
    ghost: {
      type: Boolean,
      default: false
    },
    disabled: {
      type: Boolean,
      default: false
    },
    loading: {
      type: Boolean,
      default: false
    },
    circle: {
      type: Boolean,
      default: false
    },
    loadingIcon: {
      type: String,
      default: 'spinner'
    },
    icon: {
      type: String,
      default: ''
    },
    size: {
      default: 'default',
      validator(value) {
        return ['small', 'default', 'large'].includes(value)
      }
    },
    textColor: {
      type: String,
      default: null
    }
  },
  data() {
    return {
      prefixCls: `${prefix}-button`,
      buttonType: 'button',
      pulsing: false
    }
  },
  computed: {
    isIconOnly() {
      return !this.$slots.default
    },
    className() {
      const {
        prefixCls,
        type,
        simple,
        ghost,
        disabled,
        loading,
        circle,
        isIconOnly,
        size,
        pulsing
      } = this

      return {
        [prefixCls]: true,
        [`${prefixCls}--${type}`]: type !== 'default',
        [`${prefixCls}--simple`]: !ghost && simple,
        [`${prefixCls}--ghost`]: ghost,
        [`${prefixCls}--disabled`]: disabled,
        [`${prefixCls}--loading`]: loading,
        [`${prefixCls}--circle`]: circle,
        [`${prefixCls}--icon-only`]: isIconOnly,
        [`${prefixCls}--${size}`]: size !== 'defalut',
        [`${prefixCls}--pulsing`]: pulsing
      }
    },
    style() {
      return {
        color: this.textColor
      }
    }
  },
  methods: {
    handleClick(event) {
      this.pulsing = false

      requestAnimationFrame(() => {
        this.pulsing = true
      })

      this.$emit('on-click', event)
    },
    test() {
      this.pulsing = false
    }
  }
}
</script>
