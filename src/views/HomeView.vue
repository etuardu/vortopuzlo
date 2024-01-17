<template>
  <div>
    <div class="cupboard" ref="cupboard">
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
    <div class="fridge" ref="fridge">
      <div
        class="slot"
        v-for="(tile, index) in [ ...fridge, { value: '' } ]"
        :key="`${tile.value}-${index}`"
        :data-ghost="!tile.value"
        :data-index="index"
      >
        <VTile
          :tile="tile"
          :index="index"
          drag-effect="move"
        ></VTile>
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
import VTile from '@/components/VTile.vue'
export default {
  components: {
    VTile
  },
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
    }))
    this.tiles = this.sort_tiles(this.tiles)
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

    fridge_el.addEventListener('drop', e => {
      console.log(e.dataTransfer.getData("text"))
      const slot_el = e.target.closest('.slot')
      if (!slot_el) return
      slot_el.classList.remove('over')

      const index = slot_el.dataset.index

      let dragData
      try {
        dragData = JSON.parse(
          e.dataTransfer.getData("text")
        )
      } catch {
        return
      }

      console.log(dragData)

      if (dragData.effect === 'move') {
        that.move_array_element(
          that.fridge,
          dragData.index,
          index
        )
      }

      if (dragData.effect === 'copy') {
        console.log(
          index,
          that.tiles[dragData.index]
        )
        that.fridge.splice(
          index,
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
      if (dragData.effect === 'move') {
        that.fridge.splice(
          dragData.index,
          1
        )
      }
    })
  },
  methods: {
    sort_tiles(tiles) {
      // sort by type and then by value alphabetically.
      const type_order = {
        ending: 0,
        prefix: 1,
        suffix: 2,
        root: 3,
      }
      return tiles.sort(
        (a, b) => {
          return type_order[a.type] - type_order[b.type]
          || a.value.localeCompare(b.value)
        }
      )
    },
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

.slot {
  display: flex;
}

.slot[data-ghost='true'] {
  flex: 1;
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


@import url('https://fonts.googleapis.com/css2?family=Roboto+Condensed:wght@700&display=swap');
.tile {
  color: var(--white);
  font-family: "Roboto Condensed", sans-serif;
  font-optical-sizing: auto;
  font-weight: 700;
  font-style: normal;
}

</style>
