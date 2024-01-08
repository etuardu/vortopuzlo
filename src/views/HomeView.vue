<template>
  <div>
    <div class="cupboard">
      <div class="tile" v-for="tile in tiles" :key="tile.value">
        {{ tile.value }}
      </div>
    </div>
    <div class="fridge">
      <div
        class="tile word-tile"
        v-for="tile in [ ...fridge, { value: '' } ]"
        :key="tile.value"
        :data-ghost="!tile.value"
      >
        {{ tile.value || '&nbsp;' }}
      </div>
    </div>
  </div>
</template>
<script>
export default {
  data() { return {
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
}
</script>
<style scoped>
.tile {
  padding: .5em .5em;
  display: inline-block;
  text-transform: uppercase;
  font-size: 18px;
  background-color: #dddddd;
  border: 2px solid #cccccc;
}
.word-tile[data-ghost='true'] {
  background-color: transparent;
  border-color: transparent;
  border-left-color: #cccccc;
}
.word-tile {
  border-top: 0;
  border-bottom: 0;
  border-right: 2px solid transparent;
}
.word-tile:hover {
  border-left: 2px solid black;
}
.cupboard {
  display: flex;
  gap: 1em;
}
.fridge {
  margin-top: 50px;
}
</style>
