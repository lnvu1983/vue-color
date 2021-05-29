<template>
  <div role="application" aria-label="Mix color picker" :class="['vc-mix', disableAlpha ? 'vc-mix__disable-alpha' : '']">
    <div class="vc-mix-header-control">
      <div class="vc-mix-color-wrap">
        <div :aria-label="`Current color is ${activeColor}`" class="vc-mix-active-color" :style="{background: activeColor}"></div>
        <checkboard></checkboard>
      </div>

      <div class="vc-mix-color-code-container">
        <div class="vc-mix-color-label">Hex #</div>

        <input id="vc-mix-color-code" :value="hex" @change="inputRgbChange" />
      </div>

      <div v-if="!disableAlpha" class="vc-mix-color-alpha-container">
        <input id="vc-mix-color-alpha" :value="alpha" :arrow-offset="0.01" :max="1" @change="inputAlphaChange" />
      </div>
    </div>

    <div class="vc-mix-main-control">
      <div class="vc-mix-saturation-wrap">
        <saturation v-model="colors" @change="childChange"></saturation>
      </div>

      <div class="vc-mix-hue-wrap">
        <hue v-model="colors" @change="childChange"  direction="vertical"></hue>
      </div>

      <div class="vc-mix-alpha-wrap" v-if="!disableAlpha">
        <alpha v-model="colors" @change="childChange" direction="vertical"></alpha>
      </div>
    </div>

    <div class="vc-mix-presets" role="group" aria-label="A color preset, pick one to set as current color">
      <Button
        class="vc-mix-presets-color vc-add-color-button"
        aria-label="Add color"
        @click="addColor">
        {{ addColorText }}
      </Button>

      <template v-for="(c, index) in presetColors">
        <div
          :key="index"
          class="vc-mix-presets-color"
          :aria-label="`Color: ${c}`"
          :data-color="c"
          @click="handlePreset(c)">
          <div
            class="vc-mix-presets-color-inner"
            :style="{backgroundColor: c}">
          </div>

          <checkboard />
        </div>
      </template>
    </div>
  </div>
</template>

<script>
import colorMixin from '../mixin/color'
import saturation from './common/Saturation.vue'
import hue from './common/Hue.vue'
import alpha from './common/Alpha.vue'
import checkboard from './common/Checkboard.vue'

const presetColors = []

export default {
  name: 'Mix',
  mixins: [colorMixin],
  components: {
    saturation,
    hue,
    alpha,
    checkboard
  },
  props: {
    presetColors: {
      type: Array,
      default () {
        return presetColors
      }
    },
    disableAlpha: {
      type: Boolean,
      default: false
    },
    addColorText: {
      type: String
    }
  },
  computed: {
    hex () {
      var hex = this.colors.hex

      return hex.replace('#', '')
    },
    activeColor () {
      var rgba = this.colors.rgba

      return 'rgba(' + [rgba.r, rgba.g, rgba.b, rgba.a].join(',') + ')'
    },
    alpha () {
      return `${(this.colors.a * 100).toFixed(0)}%`
    }
  },
  methods: {
    handlePreset (c) {
      this.colorChange({
        hex: c,
        source: 'hex'
      })
    },
    childChange (data) {
      this.colorChange(data)
    },
    inputRgbChange (e) {
      const hex = e.target.value

      if (!this.isValidRgb(hex)) {
        e.target.value = this.hex

        return
      }

      const opacity = document.getElementById('vc-mix-color-alpha').value

      this.rgbChange(hex)

      this.opacityChange(opacity)
    },
    inputAlphaChange (e) {
      const opacity = e.target.value

      if (!this.isValidOpacity(opacity)) {
        e.target.value = this.alpha

        return
      }

      this.opacityChange(opacity)
    },
    isValidRgb (hex) {
      return this.isValidHex(hex) && (hex.length === 3 || hex.length === 6)
    },
    rgbChange (hex) {
      this.colorChange({
        hex: hex,
        source: 'hex'
      })
    },
    isValidOpacity (opacity) {
      const regex = new RegExp(/^[\d]{1,3}[%]{0,1}$/g)

      return regex.test(opacity) && parseInt(opacity, 10) <= 100
    },
    opacityChange (opacity) {
      const a = parseInt(opacity, 10) / 100

      this.colorChange({
        r: this.colors.rgba.r,
        g: this.colors.rgba.g,
        b: this.colors.rgba.b,
        a: a,
        source: 'rgba'
      })
    },
    addColor () {
      this.$emit('addPreset', this.colors.hex8)
    }
  }
}
</script>

<style>
.vc-mix {
  position: relative;
  min-width: 305px;
  padding: 5px;
}

.vc-mix-header-control {
  position: relative;
  display: flex;
  width: 100%;
  justify-content: space-between;
}

.vc-mix-header-control .vc-mix-color-wrap {
  position: relative;
  min-width: 40px;
  max-width: 40px;
  height: 40px;
  border: 1px solid gray;
}

.vc-mix-header-control .vc-mix-color-wrap .vc-mix-active-color,
.vc-mix-header-control .vc-mix-color-wrap .vc-checkerboard {
  min-width: 38px;
  max-width: 38px;
  height: 38px;
}

.vc-mix-header-control .vc-mix-color-wrap .vc-mix-active-color {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 2;
}

.vc-mix-header-control .vc-mix-color-wrap .vc-checkerboard {
  background-size: cover;
}

.vc-mix-header-control .vc-mix-color-code-container {
  width: 100%;
  display: flex;
  padding: 7px 0;
}

.vc-mix-header-control .vc-mix-color-code-container .vc-mix-color-label {
  display: flex;
  align-items: center;
  justify-content: flex-end;
  width: 37px;
  margin-left: 11px;
  text-align: right;
  border: 1px solid gray;
  border-right: none;
}

.vc-mix-header-control .vc-mix-color-code-container #vc-mix-color-code {
  width: 63px;
  outline: none;
  border: 1px solid gray;
  border-left: none;
}

.vc-mix-header-control .vc-mix-color-alpha-container {
  display: flex;
  min-width: 48px;
  max-width: 48px;
  padding: 7px 0;
}

.vc-mix-header-control .vc-mix-color-alpha-container #vc-mix-color-alpha {
  width: 100%;
  text-align: center;
  outline: none;
  border: 1px solid gray;
}

.vc-mix-main-control {
  display: flex;
  justify-content: space-between;
  height: 240px;
  margin-top: 7px;
}

.vc-mix-main-control .vc-mix-saturation-wrap,
.vc-mix-main-control .vc-mix-hue-wrap .vc-hue,
.vc-mix-main-control .vc-mix-alpha-wrap {
  border-radius: 2px;
}

.vc-mix-main-control .vc-mix-saturation-wrap,
.vc-mix-main-control .vc-mix-hue-wrap,
.vc-mix-main-control .vc-mix-hue-wrap .vc-hue,
.vc-mix-main-control .vc-mix-alpha-wrap {
  position: relative;
  height: 100%;
  overflow: hidden;
}

.vc-mix-main-control .vc-mix-saturation-wrap {
  width: 240px;
}

.vc-mix-main-control .vc-mix-hue-wrap,
.vc-mix-main-control .vc-mix-alpha-wrap {
  width: 16px;
}

.vc-hue,
.vc-alpha {
  width: 100%;
  height: 100%;
}

.vc-alpha .vc-alpha-checkboard-wrap .vc-checkerboard {
  background-repeat: repeat-y;
}

.vc-alpha .vc-alpha-gradient {
  height: 100%;
  overflow: hidden;
}

.vc-hue .vc-hue-container,
.vc-alpha .vc-alpha-container {
  width: 100%;
  margin: 0;
}

.vc-hue .vc-hue-container .vc-hue-pointer,
.vc-alpha .vc-alpha-container .vc-alpha-pointer {
  width: 100%;
  padding: 0 2px;
}

.vc-hue .vc-hue-container .vc-hue-pointer .vc-hue-picker,
.vc-alpha .vc-alpha-container .vc-alpha-pointer .vc-alpha-picker {
  width: 100%;
  margin: 0;
  transform: none;
}

.vc-mix-presets {
  margin-top: 10px;
}

.vc-mix-presets-color {
  cursor: pointer;
  position: relative;
  margin-top: 5px;
  margin-right: 2px;
  float: left;
  width: 28px;
  height: 28px;
  border: 1px solid gray;
}

.vc-mix-presets-color .vc-mix-presets-color-inner {
  position: absolute;
  top: 0px;
  right: 0px;
  bottom: 0px;
  left: 0px;
  z-index: 2;
}

.vc-mix-presets-color .vc-checkerboard {
  position: relative;
  width: 26px;
  height: 26px;
  background-size: cover;
}
</style>
