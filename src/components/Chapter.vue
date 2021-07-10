<template>
  <q-img :src="src" loading="lazy" spinner-color="white" v-if="show">
    <template v-slot:loading>
      <div class="absolute-full flex flex-center text-white">
        <q-circular-progress
          indeterminate
          size="50px"
          color="primary"
          class="q-ma-md"
        />
      </div>
    </template>
    <template v-slot:error>
      <div class="absolute-full flex flex-center text-white">
        <q-btn round color="primary" icon="refresh" v-on:click="refresh" />
      </div>
    </template>
  </q-img>
</template>

<script>
export default {
  name: "Chapter",
  props: {
    src: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      loading: true,
      show: true,
    };
  },
  methods: {
    refresh() {
      this.show = false;
      this.show = true;
    },
  },
};
</script>
