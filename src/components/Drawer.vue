<template>
  <q-list>
    <q-item-label header class="text-grey-8"> Navigation </q-item-label>

    <NavLink v-for="link in navLinks" :key="link.title" v-bind="link" />

    <!-- <q-item>
      <q-btn-dropdown color="primary" icon="settings" label="Settings">
        <div class="row no-wrap q-pa-md">
          <div class="column">
            <div class="text-h6 q-mb-md">Settings</div>
            <q-select
              v-model="language"
              :options="language_options"
              label="Language"
            />
          </div>
        </div>
      </q-btn-dropdown>
    </q-item> -->
  </q-list>
</template>

<script>
import { defineComponent } from "vue";
import NavLink from "components/NavLink.vue";

export default defineComponent({
  name: "Drawer",
  components: {
    NavLink,
  },
  data() {
    return {
      language_options: ["en", "es"],
      navLinks: [
        {
          title: "Home",
          icon: "home",
          link: "/",
        },
        {
          title: "Search",
          icon: "search",
          link: "/search",
        },
        {
          title: "Quasar Github",
          caption: "github.com/quasarframework",
          icon: "code",
          link: "https://github.com/quasarframework",
        },
      ],
    };
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
});
</script>
