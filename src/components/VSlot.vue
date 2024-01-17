<template>
  <div
    :class="{
      slot: true,
      over: over,
      ghost: ghost,
      caret: ghost && index === 0,
    }"
    @dragenter="dragenter"
    @dragleave="dragleave"
    @dragover="(e) => e.preventDefault()"
    @drop="drop"
  >
    <slot></slot>
  </div>
</template>
<script>
export default {
  props: [
    'index', 'ghost', 'dragging_index'
  ],
  data: () => ({
    over: false
  }),
  methods: {
    parseDragData(e) {
      try {
        return JSON.parse(
          e.dataTransfer.getData("text")
        )
      } catch {
        return null
      }
    },
    dragenter() {
      if (this.dragging_index === this.index) return
      this.over = true
    },
    dragleave() {
      this.over = false
    },
    drop(e) {
      e.preventDefault()
      this.over = false
      const dragData = this.parseDragData(e)
      if (!dragData) return
      this.$emit(
        'dropped',
        dragData.effect,
        dragData.index,
        this.index
      )
    }
  }
}
</script>
<style scoped>
.slot {
  display: flex;
}
.slot::before {
  content: '';
  display: block;
  background: #999;
  width: 0px;
  border-radius: 2px;
  height: 100%;
  margin: 0 0;
  transition: margin .1s ease-in;
}
.slot.over::before {
  width: 3px;
  margin: 0 .25em;
}
.slot.caret::before {
  width: 3px;
}
.slot.ghost {
  flex: 1;
}
</style>
<style>
.slot.over *,
.slot.ghost * {
  pointer-events: none;
  /* https://stackoverflow.com/a/18582960/440172 */
}
</style>
