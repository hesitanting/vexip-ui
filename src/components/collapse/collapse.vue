<template>
  <div :class="className">
    <slot></slot>
  </div>
</template>

<script>
import { removeArrayItem } from '../../utils/common'

const { prefix } = require('../../style/basis/variable')

export default {
  name: 'Collapse',
  props: {
    expanded: {
      type: [String, Number, Array],
      default: null
    },
    card: {
      type: Boolean,
      default: false
    },
    accordion: {
      type: Boolean,
      default: false
    },
    arrowType: {
      default: 'right',
      validator(value) {
        return ['right', 'left', 'none'].includes(value)
      }
    }
  },
  data() {
    const { expanded, accordion } = this

    let currentExpanded

    if (accordion && Array.isArray(expanded)) {
      currentExpanded = expanded[0]
    } else {
      currentExpanded = expanded || expanded === 0 ? expanded : []
      currentExpanded = Array.isArray(currentExpanded)
        ? Array.from(currentExpanded)
        : [currentExpanded]
    }

    return {
      prefix: `${prefix}-collapse`,
      items: [],
      currentExpanded
    }
  },
  computed: {
    className() {
      const { prefix, card, arrowType } = this

      return [
        prefix,
        {
          [`${prefix}--card`]: card
        },
        `${prefix}--arrow-${arrowType}`
      ]
    }
  },
  watch: {
    expanded(value) {
      this.currentExpanded = value
    },
    currentExpanded(value) {
      this.updateItemExpanded()
      this.$emit('on-change', value)
    }
  },
  mounted() {
    this.$nextTick(this.updateItemExpanded)
  },
  methods: {
    toggleExpandItem(label, expanded) {
      if (!label && label !== 0) return

      if (this.accordion) {
        this.currentExpanded = expanded ? [label] : []
      } else {
        if (!Array.isArray(this.currentExpanded)) {
          this.currentExpanded = [this.currentExpanded]
        }

        if (expanded) {
          this.currentExpanded.push(label)
        } else {
          removeArrayItem(this.currentExpanded, label)
        }
      }
    },
    updateItemExpanded() {
      let expanded = this.currentExpanded

      expanded = Array.isArray(expanded) ? expanded : [expanded]

      this.items.forEach(item => {
        item.currentExpanded = expanded.includes(item.currentLabel)
      })
    }
  }
}
</script>
