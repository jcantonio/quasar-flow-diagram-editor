<template>
  <div class="node" :class="{ selected: selected }" :style="style">
    <header :style="headerStyle">
      <q-icon class="icon" size="xs" :name="icon"/>
      {{ title }}
      <q-btn class="delete" round size="xs" icon="remove" @click="deleteNode"/>
    </header>
    <div class="inputs">
      <div class="input" v-bind:key="index" v-for="(slot, index) in inputs">
        <div
          class="circle inputSlot"
          :class="{ active: slot.active }"
          @mouseup="slotMouseUp($event, index)"
          @mousedown="slotBreak($event, index)"
        ></div>
        {{ slot.label }}
      </div>
    </div>
    <div class="outputs">
      <div class="output" v-bind:key="index" v-for="(slot, index) in outputs">
        <div
          class="circle"
          :class="{ active: slot.active }"
          @mousedown="slotMouseDown($event, index)"
        ></div>
        {{ slot.label }}
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Node',
  props: {
    x: {
      type: Number,
      default: 0,
      validator: function (val) {
        return typeof val === 'number'
      }
    },
    y: {
      type: Number,
      default: 0,
      validator: function (val) {
        return typeof val === 'number'
      }
    },
    selected: Boolean,
    title: {
      type: String,
      default: 'Title'
    },
    icon: {
      type: String,
      default: 'adjust'
    },
    inputs: Array,
    outputs: Array,

    options: {
      type: Object
    }
  },
  created () {
    this.mouseX = 0
    this.mouseY = 0

    this.lastMouseX = 0
    this.lastMouseY = 0

    this.linking = false
    this.dragging = false
  },
  mounted () {
    document.documentElement.addEventListener('mousemove', this.handleMove, true)
    document.documentElement.addEventListener('mousedown', this.handleDown, true)
    document.documentElement.addEventListener('mouseup', this.handleUp, true)
  },
  beforeDestroy () {
    document.documentElement.removeEventListener('mousemove', this.handleMove, true)
    document.documentElement.removeEventListener('mousedown', this.handleDown, true)
    document.documentElement.removeEventListener('mouseup', this.handleUp, true)
  },
  data () {
    return {
      width: this.options.width,
      hasDragged: false
    }
  },
  methods: {
    handleMove (e) {
      this.mouseX = e.pageX || e.clientX + document.documentElement.scrollLeft
      this.mouseY = e.pageY || e.clientY + document.documentElement.scrollTop

      if (this.dragging && !this.linking) {
        let diffX = this.mouseX - this.lastMouseX
        let diffY = this.mouseY - this.lastMouseY

        this.lastMouseX = this.mouseX
        this.lastMouseY = this.mouseY

        this.moveWithDiff(diffX, diffY)

        this.hasDragged = true
      }
    },
    handleDown (e) {
      this.mouseX = e.pageX || e.clientX + document.documentElement.scrollLeft
      this.mouseY = e.pageY || e.clientY + document.documentElement.scrollTop

      this.lastMouseX = this.mouseX
      this.lastMouseY = this.mouseY

      const target = e.target || e.srcElement
      if (this.$el.contains(target) && e.which === 1) {
        this.dragging = true

        this.$emit('select')

        if (e.preventDefault) e.preventDefault()
      }
    },
    handleUp () {
      if (this.dragging) {
        this.dragging = false

        if (this.hasDragged) {
          this.save()
          this.hasDragged = false
        }
      }

      if (this.linking) {
        this.linking = false
      }
    },
    // Slots
    slotMouseDown (e, index) {
      this.linking = true

      this.$emit('linkingStart', index)
      if (e.preventDefault) e.preventDefault()
    },
    slotMouseUp (e, index) {
      this.$emit('linkingStop', index)
      if (e.preventDefault) e.preventDefault()
    },
    slotBreak (e, index) {
      this.linking = true

      this.$emit('linkingBreak', index)
      if (e.preventDefault) e.preventDefault()
    },
    save () {
      this.$emit('update')
    },
    deleteNode () {
      this.$emit('delete')
    },
    moveWithDiff (diffX, diffY) {
      let left = this.x + diffX / this.options.scale
      let top = this.y + diffY / this.options.scale

      this.$emit('update:x', left)
      this.$emit('update:y', top)
    }
  },
  computed: {
    style () {
      return {
        top: this.options.center.y + this.y * this.options.scale + 'px',
        left: this.options.center.x + this.x * this.options.scale + 'px',
        width: this.width + 'px',
        transform: 'scale(' + (this.options.scale + '') + ')',
        transformOrigin: 'top left'
      }
    },
    headerStyle () {
      return {
        height: this.options.titleHeight + 'px'
      }
    }
  }
}
</script>

<style lang="less" scoped>
@nodeBorder: 1px;

@ioPaddingInner: 2px 0;
@ioHeight: 16px;
@ioFontSize: 14px;

@circleBorder: 1px;
@circleSize: 10px;
@circleMargin: 2px; // left/right

@circleNewColor: #00b7ff;
@circleRemoveColor: #af3b3b;
@circleConnectedColor: #e9e9e9;

.node {
  position: absolute;
  box-sizing: border-box;
  border: @nodeBorder solid rgb(218, 216, 216);
  background: white;
  z-index: 1;
  opacity: 0.9;
  cursor: move;

  &.selected {
    box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
    z-index: 2;
  }

  > header {
    background: #e6e6e6;
    text-align: center;
    > .delete {
      float: right;
      position: absolute;
      top: -5px;
      right: -5px;
    }
  }

  .icon {
      float: left;
  }
  .inputs,
  .outputs {
    padding: @ioPaddingInner;

    display: node;
    width: 50%;

    > * {
      width: 100%;
    }
  }

  .circle {
    box-sizing: border-box;
    margin-top: @ioHeight / 2 - @circleSize / 2;

    width: @circleSize;
    height: @circleSize;

    border: @circleBorder solid rgba(0, 0, 0, 0.5);
    border-radius: 100%;

    cursor: crosshair;
    &.active {
      background: @circleConnectedColor;
    }
  }

  .inputs {
    float: left;
    text-align: left;

    margin-left: -(@circleSize / 2 + @nodeBorder);
  }

  .input,
  .output {
    height: @ioHeight;
    overflow: hidden;
    font-size: @ioFontSize;

    &:last-child {
    }
  }

  .input {
    float: left;

    .circle {
      float: left;
      margin-right: @circleMargin;

      &:hover {
        background: @circleNewColor;

        &.active {
          background: @circleRemoveColor;
        }
      }
    }
  }

  .outputs {
    float: right;
    text-align: right;

    margin-right: -(@circleSize / 2 + @nodeBorder);
  }

  .output {
    float: right;

    .circle {
      float: right;
      margin-left: @circleMargin;

      &:hover {
        background: @circleNewColor;
      }
    }
  }
}
</style>
