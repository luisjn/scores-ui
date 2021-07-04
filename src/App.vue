<template>
  <v-app>
    <v-app-bar app color="primary" dark>
      <v-toolbar-title href="/" class="display-2">Snake</v-toolbar-title>

      <v-spacer></v-spacer>

      <v-btn
        href="/play"
        text
        outlined
      >
        <span class="mr-2">play</span>
        <v-icon>mdi-gamepad-variant-outline</v-icon>
      </v-btn>
      <v-btn
        href="/scores"
        text
        outlined
      >
        <span class="mr-2">scores</span>
        <v-icon>mdi-format-list-numbered</v-icon>
      </v-btn>
    </v-app-bar>

    <v-main>
      <router-view />
    </v-main>
  </v-app>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "App",

  data: () => ({
    //
  }),
});
</script>
