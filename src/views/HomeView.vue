<template>
  <div>
    <div class="cupboard" ref="cupboard">
      <div
        class="drawer"
      >
        <div
          class="tile"
          v-for="tile in sorted_tiles"
          :key="tile.value"
          draggable="true"
          :data-value="tile.value"
          :data-type="tile.type"
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
        class="slot"
        v-for="(tile, index) in [ ...fridge, { value: '' } ]"
        :key="`${tile.value}-${index}`"
        :data-ghost="!tile.value"
      >
        <div
          class="tile word-tile"
          :draggable="!!tile.value"
          :data-value="tile.value"
          :data-type="tile.type"
          :data-index="index"
          :data-explanation="tile.it"
          :title="tile.it"
        >
          {{ tile.value || '&nbsp;' }}
        </div>
      </div>
    </div>
    <div class="translation">
      <template v-if="fridge_word">
        🇮🇹
        <template v-if="translation">
          {{ translation }}
        </template>
        <template v-else>
          <i>Traduzione non disponibile</i>
        </template>
      </template>
      <template v-else>
        <i>Trascina una particella per iniziare</i>
      </template>
    </div>
  </div>
</template>
<script>
export default {
  data() { return {
    current_drag_tile: null,
    tiles: [
    /*
      { type: 'prefix', value: 'mal', it: 'Contrario' },

      { type: 'suffix', value: 'in', it: 'Femminile' },
      { type: 'suffix', value: 'ej', it: 'Luogo' },
      { type: 'suffix', value: 'estr', it: 'Capo' },
      { type: 'suffix', value: 'ul', it: 'Individuo' },

      { type: 'root',   value: 'san', it: 'Salute' },

      { type: 'ending', value: 'o', it: 'Sostantivo' },
      { type: 'ending', value: 'a', it: 'Aggettivo' },
      { type: 'ending', value: 'e', it: 'Avverbio' },
      { type: 'ending', value: 'i', it: 'Verbo (infinito)' },
    */
    ],
    dictionary: {
      it: {
        /*
        "san'o": 'Salute',
        'sana': 'Sano',
        'sanulo': 'Sano (sost.)',
        'sanulino': 'Sano (f., sost.)',
        'malsano': 'Malattia',
        'malsana': 'Malato (agg.)',
        'malsani': 'Essere malato',
        'malsanulo': 'Malato (sost.)',
        'malsanulino': 'Malata (sost.)',
        'malsanulejo': 'Ospedale',
        'ulo': 'Tizio',
        'malsanulejestro': "Direttore dell'ospedale",
        'malsanulejestrino': "Direttrice dell'ospedale",
        'estro': 'Capo',
        'estrino': 'Capo (f.)',
        'ejo': 'Posto',
        'malo': 'Contrario',
        'mala': 'Opposto',
        'male': 'Contrariamente',
        'estri': 'Dirigere',
        'sane': 'In modo sano, salubremente'
        */
      }
    },
    fridge: [
    ],
  }},
  computed: {
    sorted_tiles() {
      // sort by type and then alphabetically.
      // An 'index' prop is added to reference the position
      // of the element in this.tiles
      const type_order = {
        ending: 0,
        prefix: 1,
        suffix: 2,
        root: 3,
      }
      return this.tiles.map((t, i) => ({
        index: i,
        ...t
      })).toSorted(
        (a, b) => {
          return type_order[a.type] - type_order[b.type]
          || a.value.localeCompare(b.value)
        }
      )
    },
    fridge_word() {
      return this.fridge.map(t => t.value).join("'")
    },
    translation() {
      return this.dictionary['it'][this.fridge_word]
    }
  },
  async created() {
    this.tiles = (await this.read_gsheet('vorteroj')).slice(
      1 // skip heading
    ).map(i => ({
      type: {'prefikso': 'prefix', 'sufikso': 'suffix', 'radiko': 'root', 'finaĵo': 'ending'}[i[0]],
      value: i[1],
      it: i[2]
    }) )
    this.dictionary = { it:
      Object.fromEntries((await this.read_gsheet('tradukoj')).slice(
        1 // skip heading
      ))
    }
  },
  mounted() {
    const fridge_el = this.$refs['fridge']
    const cupboard_el = this.$refs['cupboard']
    const that = this

    fridge_el.addEventListener('dragenter', e => {
      const slot_el = e.target.closest('.slot')
      if (!slot_el) return
      console.log("dragenter slot", slot_el.innerText)
      const tile_el = slot_el.querySelector('.tile')
      if (tile_el === that.current_drag_tile) return
      slot_el.classList.add('over')
    })

    fridge_el.addEventListener('dragleave', e => {
      const slot_el = e.target.closest('.slot')
      if (!slot_el) return
      console.log("dragleave slot", slot_el.innerText)
      const tile_el = slot_el.querySelector('.tile')
      if (tile_el === that.current_drag_tile) return
      slot_el.classList.remove('over')
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
      const slot_el = e.target.closest('.slot')
      if (!slot_el) return
      slot_el.classList.remove('over')

      const tile_el = slot_el.querySelector('.tile')

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
    },
    async read_gsheet(sheet_name) {
      const doc_id = '1yeMoKfNRDUc8BzWSf_-54oBqmzCnOtL-nztDLgWCoTY'
      const url = (doc_id, sheet_name) => `https://docs.google.com/spreadsheets/d/${doc_id}/gviz/tq?tqx=out:json&sheet=${sheet_name}`
      const response = await (await fetch(url(doc_id, sheet_name))).text()
      return JSON.parse(
        response.slice(
          response.indexOf("{"),
          response.lastIndexOf("}") + 1
        )
      ).table.rows.map(e=>e.c.map(e=>e.v))
    },
  }
}
</script>
<style>

body, html {
  margin: 0;
  padding: 0;
}

.tile {
  display: inline-block;
  text-transform: uppercase;
  font-size: 24px;
  position: relative;
  padding: .5em;
  background: #ddd;
  margin: .1em;
  border-radius: 4px;
  /* box-shadow: 0 4px 0 #ccc; */
}

.slot {
  display: flex;
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

.slot[data-ghost='true'] {
  flex: 1;
}
.slot[data-ghost='true'] .word-tile {
  box-shadow: none;
  background-color: transparent;
}
.slot[data-ghost='true']:first-child::before {
  width: 3px;
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
  padding: 25px;
  background: var(--cream);
}
.fridge {
  padding-top: 20px;
  margin-top: 50px;
  overflow-x: auto;
  padding-bottom: 3px;
  display: flex;
  margin: 20px;
}
.translation {
  margin: 20px;
  margin-left: 30px;
  font-size: 26px;
  font-family: "Roboto Condensed", sans-serif;
}
.translation i {
  opacity: .25;
}

.over *,
.slot[data-ghost='true'] * {
  pointer-events: none;
  /* https://stackoverflow.com/a/18582960/440172 */
}

:root {
  --red: #c1384a;
  --orange: #dc9136;
  --blue: #4c8694;
  --green: #64b39e;
  --black: #3a3531;
  --cream: #f8f2e2;
  --white: #fff;
}

.tile[data-type="prefix"] { background-color: var(--green); }
.tile[data-type="suffix"] { background-color: var(--orange); }
.tile[data-type="ending"] { background-color: var(--red); }
.tile[data-type="root"] { background-color: var(--blue); }

@import url('https://fonts.googleapis.com/css2?family=Roboto+Condensed:wght@700&display=swap');
.tile {
  color: var(--white);
  font-family: "Roboto Condensed", sans-serif;
  font-optical-sizing: auto;
  font-weight: 700;
  font-style: normal;
}

</style>
