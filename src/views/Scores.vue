<template>
  <scoreboard />
</template>

<script lang="ts">
import Scoreboard from "../components/Scoreboard.vue";
import Vue from "vue";

export default Vue.extend({
  name: "Scores",

  components: {
    Scoreboard,
  },
});
</script>