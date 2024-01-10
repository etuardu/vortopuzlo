<template>
  <div>
    <div class="cupboard" ref="cupboard">
      <div
        class="drawer"
        v-for="type in ['prefix', 'root', 'suffix', 'ending']"
        :key="type"
      >
        <div
          class="tile"
          v-for="tile in indexed_tiles_by_type[type]"
          :key="tile.value"
          draggable="true"
          :data-value="tile.value"
          :data-index="tile.index"
          :data-explanation="tile.it"
          :title="tile.it"
        >
          {{ tile.value }}
        </div>
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
        :data-explanation="tile.it"
        :title="tile.it"
      >
        {{ tile.value || '&nbsp;' }}
      </div>
    </div>
    <div class="translation">
      {{ translation }}
    </div>
  </div>
</template>
<script>
export default {
  data() { return {
    current_drag_tile: null,
    tiles: [
      { type: 'prefix', value: 'mal', it: 'Contrario' },
      { type: 'prefix', value: 'mal', it: 'Contrario' },
      { type: 'prefix', value: 'mal', it: 'Contrario' },
      { type: 'prefix', value: 'mal', it: 'Contrario' },
      { type: 'prefix', value: 'mal', it: 'Contrario' },
      { type: 'prefix', value: 'mal', it: 'Contrario' },
      { type: 'prefix', value: 'mal', it: 'Contrario' },
      { type: 'prefix', value: 'mal', it: 'Contrario' },
      { type: 'prefix', value: 'mal', it: 'Contrario' },

      { type: 'suffix', value: 'in', it: 'Femminile' },
      { type: 'suffix', value: 'ej', it: 'Luogo' },
      { type: 'suffix', value: 'estr', it: 'Capo' },
      { type: 'suffix', value: 'ul', it: 'Individuo' },
      { type: 'suffix', value: 'ul', it: 'Individuo' },
      { type: 'suffix', value: 'ul', it: 'Individuo' },
      { type: 'suffix', value: 'ul', it: 'Individuo' },
      { type: 'suffix', value: 'ul', it: 'Individuo' },

      { type: 'root',   value: 'san', it: 'Salute' },
      { type: 'root',   value: 'san', it: 'Salute' },
      { type: 'root',   value: 'san', it: 'Salute' },
      { type: 'root',   value: 'san', it: 'Salute' },
      { type: 'root',   value: 'san', it: 'Salute' },
      { type: 'root',   value: 'san', it: 'Salute' },
      { type: 'root',   value: 'san', it: 'Salute' },
      { type: 'root',   value: 'san', it: 'Salute' },

      { type: 'ending', value: 'o', it: 'Sostantivo' },
      { type: 'ending', value: 'a', it: 'Aggettivo' },
      { type: 'ending', value: 'e', it: 'Avverbio' },
      { type: 'ending', value: 'i', it: 'Verbo (infinito)' },
    ],
    dictionary: {
      it: {
        'sano': 'salute',
        'malsano': 'malattia',
        'malsanulo': 'malato',
        'malsanulino': 'malata',
        'malsanulejo': 'ospedale',
      }
    },
    fridge: [
    ],
  }},
  computed: {
    indexed_tiles_by_type() {
      const ret = {}
      for (let i=0; i<this.tiles.length; i++) {
        const tile = this.tiles[i]
        ret[tile.type] = [
          ...(ret[tile.type] ?? []),
          { index: i, ...tile }
        ]
      }
      return ret
    },
    fridge_word() {
      return this.fridge.map(t => t.value).join("")
    },
    translation() {
      return this.dictionary['it'][this.fridge_word]
    }
  },
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
      tile_el.classList.add('dragging')
      e.dataTransfer.effectAllowed = "move"
      e.dataTransfer.setData("text", JSON.stringify({
        action: 'move',
        index: tile_el.dataset.index,
      }));
    })

    fridge_el.addEventListener('dragend', e => {
      that.current_drag_tile = null
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      tile_el.classList.remove('dragging')
    })

    cupboard_el.addEventListener('dragstart', e => {
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      tile_el.classList.add('dragging')
      e.dataTransfer.effectAllowed = "copy"
      e.dataTransfer.setData("text", JSON.stringify({
        action: 'add',
        index: tile_el.dataset.index,
      }));
    })
    cupboard_el.addEventListener('dragend', e => {
      that.current_drag_tile = null
      const tile_el = e.target.closest('.tile')
      if (!tile_el) return
      tile_el.classList.remove('dragging')
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
  position: relative;
}
/*
.tile:not(.dragging):not([data-ghost='true']):hover::after {
  position: absolute;
  content: attr(data-explanation);
  display: block;
  height: fit-content;
  text-transform: initial;
  font-size: initial;
  border: 1px solid black;
  background: #fff;
  z-index: 99;
  top: calc(100% - .5em);
  left: .5em;
  padding: .5em;
}
*/

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
.fridge .dragging {
  opacity: .25;
}
.drawer .tile {
  margin: .1em;
}
.drawer {
  overflow-y: scroll;
  max-height: 50vh;
}
.cupboard {
  display: grid;
  grid-template-columns: 1fr 2fr 1fr auto;
}
.fridge {
  margin-top: 50px;
}
.cupboard, .fridge {
  margin: 20px;
}
.translation {
  margin: 20px;
  text-transform: capitalize;
  font-size: 18px;
}
</style>
