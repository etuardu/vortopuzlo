<template>
  <div>
    <VCupboard
      :tiles="tiles"
      @dropped="onCupboardDropped"
    ></VCupboard>
   <VFridge
    :fridge="fridge"
    :dragging_index="dragging_index"
    @dragging="(i) => dragging_index = i"
    @dropped="onFridgeDropped"
   >
   </VFridge>
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
import VCupboard from '@/components/VCupboard.vue'
import VFridge from '@/components/VFridge.vue'
export default {
  components: {
    VCupboard,
    VFridge,
  },
  data() { return {
    dragging_index: null,
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
  methods: {
    onCupboardDropped(effect, index) {
      // fridge tile at <index> dropped into
      // the cupboard: remove the item
      this.dragging_index = null
      if (effect === 'move') {
        this.fridge.splice(
          index,
          1
        )
      }
    },
    onFridgeDropped(effect, from_index, to_index) {
      this.dragging_index = null
      if (effect === 'move') {
        this.move_array_element(
          this.fridge,
          from_index,
          to_index,
        )
      }
      if (effect === 'copy') {
        this.fridge.splice(
          to_index,
          0,
          this.tiles[from_index]
        )
      }
    },
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
@import url('https://fonts.googleapis.com/css2?family=Roboto+Condensed:ital,wght@0,100..900;1,100..900&display=swap');

:root {
  --red: #c1384a;
  --orange: #dc9136;
  --blue: #4c8694;
  --green: #64b39e;
  --black: #3a3531;
  --cream: #f8f2e2;
  --white: #fff;
}

body, html {
  margin: 0;
  padding: 0;
}

.translation {
  margin: 20px;
  margin-left: 30px;
  font-size: 26px;
  font-family: "Roboto Condensed", sans-serif;
}
.translation i {
  opacity: .25;
  font-style: italic;
}

</style>
