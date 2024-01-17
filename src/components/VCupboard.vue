<template>
  <div
    class="cupboard"
    ref="cupboard"
    @dragover="(e) => e.preventDefault()"
    @drop="drop"
  >
    <div
      class="drawer"
    >
      <VTile
        v-for="(tile, index) in tiles"
        :tile="tile"
        :index="index"
        drag-effect="copy"
        :key="tile.value"
      >
      </VTile>
    </div>
  </div>
</template>
<script>
import VTile from '@/components/VTile.vue'
export default {
  components: {
    VTile,
  },
  props: [
    'tiles',
  ],
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
    drop(e) {
      this.dragging_index = null

      const dragData = this.parseDragData(e)
      if (!dragData) return

      this.$emit('dropped', dragData.effect, dragData.index)

    }
  }
}
</script>
<style scoped>
.drawer {
  margin: .1em;
  overflow-y: scroll;
  max-height: 50vh;
}
.cupboard {
  padding: 25px;
  background: var(--cream);
}
</style>
