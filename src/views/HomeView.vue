<template>
  <div>
    <div class="cupboard" ref="cupboard">
      <div
        class="tile"
        v-for="(tile, index) in tiles"
        :key="tile.value"
        draggable="true"
        :data-value="tile.value"
        :data-index="index"
      >
        {{ tile.value }}
      </div>
    </div>
    <div class="fridge" ref="fridge">
      <div
        class="tile word-tile"
        v-for="(tile, index) in [ ...fridge, { value: '' } ]"
        :key="`${tile.value}-${index}`"
        :data-ghost="!tile.value"
        :draggable="!!tile.value"
        :data-value="tile.value"
        :data-index="index"
      >
        {{ tile.value || '&nbsp;' }}
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() { return {
    current_drag_tile: null,
    tiles: [
      { value: 'mal', eo: 'kontraŭo' },
      { value: 'san', eo: 'medicina bonfarto' },
      { value: 'ul', eo: 'individuo' },
      { value: 'ej', eo: 'loko' },
      { value: 'estr', eo: 'ĉefo' },
      { value: 'in', eo: 'naskipova genro' },
      { value: 'o', eo: 'substantivo' },
    ],
    fridge: [
    ],
  }},
  mounted() {
    const fridge_el = this.$refs['fridge']
    const cupboard_el = this.$refs['cupboard']
    const that = this

    fridge_el.addEventListener('dragenter', e => {
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      if (tile_el === that.current_drag_tile) return
      tile_el.classList.add('over')
    })
    fridge_el.addEventListener('dragleave', e => {
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      tile_el.classList.remove('over')
    })
    fridge_el.addEventListener('dragover', e => {
      e.preventDefault()
    })

    fridge_el.addEventListener('dragstart', e => {
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      that.current_drag_tile = tile_el
      console.log(that.current_drag_tile)
      tile_el.classList.add('invisible')
      e.dataTransfer.dropEffect = "move"
      e.dataTransfer.setData("text", JSON.stringify({
        action: 'move',
        index: tile_el.dataset.index,
      }));
    })

    fridge_el.addEventListener('dragend', e => {
      that.current_drag_tile = null
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      tile_el.classList.remove('invisible')
    })

    cupboard_el.addEventListener('dragstart', e => {
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      e.dataTransfer.dropEffect = "copy"
      e.dataTransfer.setData("text", JSON.stringify({
        action: 'add',
        index: tile_el.dataset.index,
      }));
    })

    fridge_el.addEventListener('drop', e => {
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      tile_el.classList.remove('over')

      let dragData
      try {
        dragData = JSON.parse(
          e.dataTransfer.getData("text")
        )
      } catch {
        return
      }

      console.log(dragData)

      if (dragData.action === 'move') {
        that.move_array_element(
          that.fridge,
          dragData.index,
          tile_el.dataset.index
        )
      }

      if (dragData.action === 'add') {
        console.log(
          tile_el.dataset.index,
          that.tiles[dragData.index]
        )
        that.fridge.splice(
          tile_el.dataset.index,
          0,
          that.tiles[dragData.index]
        )
      }
    })

    cupboard_el.addEventListener('dragover', e => {
      e.preventDefault()
    })
    cupboard_el.addEventListener('drop', e => {
      // tile dropped from the fridge into the cupboard:
      // remove the tile

      let dragData
      try {
        dragData = JSON.parse(
          e.dataTransfer.getData("text")
        )
      } catch {
        return
      }
      if (dragData.action === 'move') {
        that.fridge.splice(
          dragData.index,
          1
        )
      }
    })
  },
  methods: {
    move_array_element(arr, src_i, dst_i) {
      console.log("move", src_i, "to", dst_i)
      if (src_i === dst_i) return
      const src = arr.splice(src_i, 1)
      if (src_i < dst_i) dst_i--
      arr.splice(dst_i, 0, src[0])
    }
  }
}
</script>
<style scoped>
.tile {
  padding: .5em .5em;
  display: inline-block;
  text-transform: uppercase;
  font-size: 24px;
  background-color: #ddd;
}
.word-tile[data-ghost='true'] {
  background-color: transparent;
  border-color: transparent;
  border-left-color: #fff;
  width: 3em;
}
.word-tile[data-ghost='true']:first-child {
  border-left-color: #ddd;
}
.word-tile {
  border-top: 0;
  border-bottom: 0;
  border-right: 5px solid transparent;
  border-left: 5px solid #fff;
}
.word-tile.over {
  border-left: 5px solid black !important;
}
.invisible {
  opacity: .25;
}
.cupboard {
  display: flex;
  gap: 1em;
}
.fridge {
  margin-top: 50px;
}
.cupboard, .fridge {
  margin: 50px;
}
</style>
