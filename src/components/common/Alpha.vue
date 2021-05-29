<template>
  <div class="vc-alpha" :class="directionClass">
    <div class="vc-alpha-checkboard-wrap">
      <checkboard></checkboard>
    </div>
    <div class="vc-alpha-gradient" :style="{background: gradientColor}"></div>
    <div class="vc-alpha-container" ref="container"
        @mousedown="handleMouseDown"
        @touchmove="handleChange"
        @touchstart="handleChange">
      <div class="vc-alpha-pointer" :style="{top: pointerTop, left: pointerLeft}">
        <div class="vc-alpha-picker">
          <div class="vc-alpha-picker-inner" />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import checkboard from './Checkboard.vue'

export default {
  name: 'Alpha',
  props: {
    value: Object,
    onChange: Function,
    direction: {
      type: String,
      // [horizontal | vertical]
      default: 'horizontal'
    }
  },
  components: {
    checkboard
  },
  data () {
    return {
      oldAlpha: 0,
      pullDirection: '',
      isElementReady: false
    }
  },
  mounted () {
    this.isElementReady = true
  },
  computed: {
    colors () {
      const a = this.value.a

      if (a !== 0 && a - this.oldAlpha > 0) this.pullDirection = 'right'
      if (a !== 0 && a - this.oldAlpha < 0) this.pullDirection = 'left'

      this.oldAlpha = a

      return this.value
    },
    gradientColor () {
      var rgba = this.colors.rgba
      var rgbStr = [rgba.r, rgba.g, rgba.b].join(',')

      const to = this.direction === 'horizontal' ? 'right' : 'top'

      return `linear-gradient(to ${to}, rgba(${rgbStr}, 0) 0%, rgba(${rgbStr}, 1) 100%)`
    },
    directionClass () {
      return {
        'vc-alpha--horizontal': this.direction === 'horizontal',
        'vc-alpha--vertical': this.direction === 'vertical'
      }
    },
    pointerTop () {
      if (this.direction !== 'vertical') return 0

      const pointerHeight = this.isElementReady ? document.querySelector('.vc-alpha-pointer').offsetHeight : 0

      const percent = this.colors.a * 100

      return `calc(${100 - percent}% - ${pointerHeight / 2}px)`
    },
    pointerLeft () {
      if (this.direction === 'vertical') return 0

      return this.colors.a * 100 + '%'
    }
  },
  methods: {
    handleChange (e, skip) {
      !skip && e.preventDefault()
      var container = this.$refs.container
      if (!container) {
        // for some edge cases, container may not exist. see #220
        return
      }
      let a = this.colors.a

      if (this.direction === 'horizontal') {
        var containerWidth = container.clientWidth

        var xOffset = container.getBoundingClientRect().left + window.pageXOffset
        var pageX = e.pageX || (e.touches ? e.touches[0].pageX : 0)
        var left = pageX - xOffset

        if (left < 0) {
          a = 0
        } else if (left > containerWidth) {
          a = 1
        } else {
          a = Math.round(left * 100 / containerWidth) / 100
        }
      } else {
        var containerHeight = container.clientHeight

        var yOffset = container.getBoundingClientRect().top + window.pageYOffset
        var pageY = e.pageY || (e.touches ? e.touches[0].pageY : 0)
        var top = pageY - yOffset

        if (top < 0) {
          a = 1
        } else if (top > containerHeight) {
          a = 0
        } else {
          a = Math.round((containerHeight - top) * 100 / containerHeight) / 100
        }
      }

      if (this.colors.a !== a) {
        this.$emit('change', {
          h: this.colors.hsl.h,
          s: this.colors.hsl.s,
          l: this.colors.hsl.l,
          a: a,
          source: 'rgba'
        })
      }
    },
    handleMouseDown (e) {
      this.handleChange(e, true)
      window.addEventListener('mousemove', this.handleChange)
      window.addEventListener('mouseup', this.handleMouseUp)
    },
    handleMouseUp () {
      this.unbindEventListeners()
    },
    unbindEventListeners () {
      window.removeEventListener('mousemove', this.handleChange)
      window.removeEventListener('mouseup', this.handleMouseUp)
    }
  }
}

</script>

<style>
.vc-alpha {
  position: absolute;
  top: 0px;
  right: 0px;
  bottom: 0px;
  left: 0px;
}
.vc-alpha-checkboard-wrap {
  position: absolute;
  top: 0px;
  right: 0px;
  bottom: 0px;
  left: 0px;
  overflow: hidden;
}
.vc-alpha-gradient {
  position: absolute;
  top: 0px;
  right: 0px;
  bottom: 0px;
  left: 0px;
}
.vc-alpha-container {
  cursor: pointer;
  position: relative;
  z-index: 2;
  height: 100%;
  margin: 0 3px;
}
.vc-alpha-pointer {
  z-index: 2;
  position: absolute;
}
.vc-alpha-picker {
  cursor: pointer;
  width: 4px;
  border-radius: 1px;
  height: 8px;
  box-shadow: 0 0 2px rgba(0, 0, 0, .6);
  background: #fff;
  margin-top: 1px;
  transform: translateX(-2px);
}
</style>
