<template>
  <div class="slider-wrapper">
    <NoSSR>
      <vue-slider
        v-model="internalValue"
        class="slider-wrapper__slider"
        :style="{ padding: '0' }"
        :rail-style="{ background: bgStyleBackground }"
        v-bind="options"
        @change="v => $emit('valueChange', v)"
      />
    </NoSSR>
  </div>
</template>

<script>
import NoSSR from 'vue-no-ssr'

const components = {
  NoSSR
}
if (process.browser) {
  const VueSlider = require('vue-slider-component')
  components['vue-slider'] = VueSlider
}

export default {
  components,
  props: {
    direction: {
      type: String,
      required: true,
      validator(value) {
        return ['horizontal', 'vertical'].includes(value)
      }
    },
    buffered: {
      type: Number,
      default: 0,
      require: true,
      validator(value) {
        return value >= 0 && value <= 1
      }
    },
    value: {
      type: Number,
      default: 0,
      require: true,
      validator(value) {
        return value >= 0 && value <= 1
      }
    }
  },
  data() {
    return {
      internalValue: this.value,
      options: {
        interval: 0.001,
        direction: this.direction === 'horizontal' ? 'ltr' : 'btt',
        width: this.direction === 'vertical' ? 4 : 'auto',
        height: this.direction === 'vertical' ? 126 : 4,
        min: 0,
        max: 1,
        tooltip: 'none',
        dotSize: 12,
        processStyle: {
          backgroundColor: '#d84939'
        }
      }
    }
  },
  computed: {
    bgStyleBackground() {
      return `linear-gradient(90deg, #7d7d7d ${this.buffered}%, black ${
        this.buffered
      }%)`
    }
  },
  watch: {
    value() {
      this.internalValue = this.value
    }
  }
}
</script>

<style lang="stylus" scoped>
.slider-wrapper
  &__slider
    cursor pointer
    & >>> .vue-slider-dot
      background-color #eeeeee
      border-radius 100%
</style>
