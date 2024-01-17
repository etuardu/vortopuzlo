<template>
  <div
    :class="classList"
    :draggable="!!tile.value"
    @mouseover="showTooltip"
    @mouseout="hideTooltip"
    @dragstart="dragstart"
    @dragend="dragend"
    ref="tile"
  >
    {{ tile.value || '&nbsp;' }}
    <div
      v-if="!dragging"
      :class="{ tooltip: true, tooltipVisible: tooltipVisible }"
      :style="{ top: `${tooltipTop}px` }"
    >{{ tile.it }}</div>
  </div>
</template>
<script>
export default {
  props: [
    'tile', 'index', 'drag-effect'
  ],
  data: () => ({
    tooltipVisible: false,
    tooltipYOffset: -5,
    dragging: false,
  }),
  computed: {
    classList() {
      const list = ['tile']
      if (this.tile.type) list.push(this.tile.type)
      if (
        this.dragging && this.dragEffect === 'move'
      ) list.push('dragging')
      return list.join(' ')
    },
  },
  methods: {
    showTooltip() {
      if (!this.tile.value) return
      const { y, height } = this.$refs['tile'].getBoundingClientRect()
      this.tooltipTop = y + window.scrollY + this.tooltipYOffset + height
      this.tooltipVisible = true
    },
    hideTooltip() {
      this.tooltipVisible = false
    },
    dragstart(e) {
      this.hideTooltip()
      e.dataTransfer.effectAllowed = this.dragEffect
      e.dataTransfer.setData("text", JSON.stringify({
        effect: this.dragEffect,
        index: this.index,
      }));
      this.dragging = true
      this.$emit('dragging', this.index)
    },
    dragend() {
      this.dragging = false
      this.$emit('dragging', null)
    }
  }
}
</script>
<style scoped>
.tooltip {
  position: absolute;
  z-index: 99;
  margin-left: -5px;
  pointer-events: none;
  text-transform: initial;
  font-size: initial;
  background: var(--black);
  color: var(--cream);
  padding: .5em;
  opacity: 0;
  transition: all .25s;
  margin-top: -10px;
}
.tooltip::after {
  position: absolute;
  display: block;
  content: '';
  width: 0;
  height: 0;
  border: 8px solid transparent;
  border-bottom-color: var(--black);
  top: -14px;
}
.tooltip.tooltipVisible {
  opacity: 1;
  margin-top: 0;
}
.tile {
  display: inline-block;
  text-transform: uppercase;
  font-size: 24px;
  padding: .5em;
  margin: .1em;
  border-radius: 4px;
  color: var(--white);
  font-family: "Roboto Condensed", sans-serif;
  font-optical-sizing: auto;
  font-weight: 700;
  font-style: normal;
}
.tile.prefix { background-color: var(--green); }
.tile.suffix { background-color: var(--orange); }
.tile.ending { background-color: var(--red); }
.tile.root { background-color: var(--blue); }
.dragging { opacity: .25; }
</style>
