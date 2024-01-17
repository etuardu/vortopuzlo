<template>
  <div class="fridge">
    <VSlot
      v-for="(tile, index) in [ ...fridge, { value: '' } ]"
      :key="`${tile.value}-${index}`"
      :index="index"
      :ghost="!tile.value"
      :dragging_index="dragging_index"
      @dropped="onSlotDropped"
    >
      <VTile
        :tile="tile"
        :index="index"
        drag-effect="move"
        @dragging="(i) => $emit('dragging', i)"
      ></VTile>
    </VSlot>
  </div>
</template>
<script>
import VSlot from '@/components/VSlot.vue'
import VTile from '@/components/VTile.vue'
export default {
  components: {
    VSlot,
    VTile,
  },
  props: [
    'fridge',
    'dragging_index',
  ],
  methods: {
    onSlotDropped(effect, from_index, to_index) {
      this.$emit('dropped', effect, from_index, to_index)
    }
  }
}
</script>
<style scoped>
.fridge {
  padding-top: 20px;
  margin-top: 50px;
  overflow-x: auto;
  padding-bottom: 3px;
  display: flex;
  margin: 20px;
}
</style>
