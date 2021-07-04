<template>
    <game />
</template>

<script>
import Vue from "vue";
import Game from "../components/Game.vue"

export default Vue.extend({
  name: "Play",

  components: {
      Game,
  },
});
</script>