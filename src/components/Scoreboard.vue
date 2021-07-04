<template>
  <div class="scores">
    <v-container>
        <v-row>
        <v-col cols="12">
            <h1 class="display-3 text-center">Scores</h1>
        </v-col>
      </v-row>
    
      <v-row>
        <v-col cols="3 offset-4">
                <v-text-field
                v-model="player"
                solo
                label="Player"
                outlined
                clearable
                ></v-text-field>
            </v-col>
            <v-col cols="1">
                <v-btn
                rounded
                color="primary"
                dark
                x-large
                @click="search"
                >
                <v-icon>mdi-magnify</v-icon>
                </v-btn>
            </v-col>
      </v-row>

      <v-row>
        <v-col cols="4 offset-4">
            <v-simple-table>
                <template v-slot:default>
                <thead>
                    <tr>
                    <th class="text-left">
                        Player
                    </th>
                    <th class="text-left">
                        Points
                    </th>
                    </tr>
                </thead>
                <tbody>
                    <tr
                    v-for="score in scores"
                    :key="score.id"
                    >
                    <td>{{ score.player }}</td>
                    <td>{{ score.points }}</td>
                    </tr>
                </tbody>
                </template>
            </v-simple-table>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script lang="ts">
import Vue from "vue";

export default Vue.extend({
  name: "Scoreboard",

  data: () => ({
    scores: [],
    player: "",
  }),
  methods: {
      search() {
          Vue.axios.get("http://localhost:8000/scores?player=" + this.player).then(res => {
            this.scores = res.data;
          }).catch(err => {
            console.log(err);
          });
      }
  },
});
</script>