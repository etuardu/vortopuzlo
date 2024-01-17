<template>
  <div
    :class="['tile', tile.type]"
    :draggable="!!tile.value"
    @mouseover="showTooltip"
    @mouseout="hideTooltip"
    @dragstart="dragstart"
    @dragend="dragend"
    ref="tile"
  >
    {{ tile.value }}
    <div
      v-if="tooltip"
      class="tooltip"
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
    tooltip: false,
    tooltipYOffset: 10,
  }),
  methods: {
    showTooltip() {
      if (!this.tile.value) return
      const { y, height } = this.$refs['tile'].getBoundingClientRect()
      this.tooltipTop = y + window.scrollY - this.tooltipYOffset + height
      this.tooltip = true
    },
    hideTooltip() {
      this.tooltip = false
    },
    dragstart(e) {
      this.hideTooltip()
      e.dataTransfer.effectAllowed = this.dragEffect
      e.dataTransfer.setData("text", JSON.stringify({
        effect: this.dragEffect,
        index: this.index,
      }));
    }
  }
}
</script>
<style scoped>
.tooltip {
  position: absolute;
  z-index: 99;
  pointer-events: none;
  text-transform: initial;
  font-size: initial;
  background: var(--black);
  color: var(--cream);
  padding: .5em;
}
.tile {
  display: inline-block;
  text-transform: uppercase;
  font-size: 24px;
  padding: .5em;
  background: #ddd;
  margin: .1em;
  border-radius: 4px;
}
.tile.prefix { background-color: var(--green); }
.tile.suffix { background-color: var(--orange); }
.tile.ending { background-color: var(--red); }
.tile.root { background-color: var(--blue); }
</style>
