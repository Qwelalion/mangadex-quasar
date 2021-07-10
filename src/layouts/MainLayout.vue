<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated>
      <q-toolbar>
        <q-btn
          flat
          dense
          round
          icon="menu"
          aria-label="Menu"
          @click="toggleLeftDrawer"
        />

        <q-toolbar-title> Mangadex Quasar </q-toolbar-title>

        <div>v0.0.1</div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="leftDrawerOpen" show-if-above bordered class="bg-grey-1">
      <Drawer />
    </q-drawer>

    <q-page-container>
      <router-view :key="$route.fullPath" />
      <!-- <router-view v-slot="{ Component }">
        <keep-alive>
          <component :is="Component" />
        </keep-alive>
      </router-view> -->
    </q-page-container>
  </q-layout>
</template>

<script>
import Drawer from "components/Drawer.vue";

import { defineComponent, ref } from "vue";

export default defineComponent({
  name: "MainLayout",

  data() {
    return {
      language_options: ["en", "es"],
    };
  },

  components: {
    Drawer,
  },

  computed: {
    language: {
      get() {
        return this.$store.state.mangadex.language;
      },
      set(val) {
        this.$store.commit("mangadex/changeLanguage", val);
      },
    },
  },

  setup() {
    const leftDrawerOpen = ref(false);

    return {
      leftDrawerOpen,
      toggleLeftDrawer() {
        leftDrawerOpen.value = !leftDrawerOpen.value;
      },
    };
  },
});
</script>
