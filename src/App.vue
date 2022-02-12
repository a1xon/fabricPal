<template>
  <v-app class="app">
    <v-main>
      <v-container :fluid="true">
        <v-row class="justify-space-around">
          <v-col cols="4" md="3">
            <Fabrics />
          </v-col>
          <v-col cols="7" md="8">
            <Pattern />
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<style>
@import "./styles/main.css";
</style>

<script>
import Fabrics from './components/Fabrics.vue'
import Footer from './components/Footer.vue'
import Pattern from './components/Pattern.vue'

import { computed, inject, provide, reactive } from "vue";

export default {
  name: 'App',

  components: {
    Fabrics,
    Footer,
    Pattern
  },

  setup() {
    const fabrics = reactive([]);
    const getFabrics = computed(() => fabrics);
    const pushFabrics = (updatedFabric) => {
      fabrics.push(...updatedFabric);
    };
    const selectFabric = (id) => {
      for (const fabric of fabrics) {
        fabric.selected = fabric.id === id;
      }
    }

    const selectedFabric = computed(() => {
      return fabrics.find(f => f.selected);
    });

    const exportObj = {
      getFabrics, selectedFabric, pushFabrics, selectFabric
    }

    for (const [name, fn] of Object.entries(exportObj)) {
      provide(name, fn);
    }

    const useProvideStore = () => {
      return {
        getFabrics: inject('getFabrics'),
        selectedFabric: inject('selectedFabric'),
        pushFabrics: inject('pushFabrics'),
        selectFabric: inject('selectFabric')
      }
    };

    provide("useProvideStore", useProvideStore);
  }

}
</script>
