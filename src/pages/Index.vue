<template>
  <q-page class="flex flex-center">
    <div class="q-ma-md" style="width: 90%; height: 100%">
      <div class="row justify-center vertical-center" v-if="loading">
        <q-circular-progress
          indeterminate
          size="50px"
          color="primary"
          class="q-ma-md"
        />
      </div>
      <div v-else>
        <q-markup-table dense>
          <thead>
            <tr>
              <th class="text-left" v-for="column in columns" :key="column">
                {{ column }}
              </th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="row in rows" :key="row.title">
              <td class="text-left">
                <q-btn
                  dense
                  style="width: 300px"
                  align="left"
                  color="white"
                  text-color="black"
                  :to="'/manga/' + row.manga.id"
                  flat
                >
                  <div class="ellipsis">
                    {{ row.manga.title }}
                  </div>
                </q-btn>
              </td>
              <td class="text-left">
                <q-btn
                  dense
                  style="width: 300px"
                  color="white"
                  align="left"
                  text-color="black"
                  :to="'/chapter/' + row.chapter.id"
                  flat
                >
                  <div class="ellipsis">
                    {{ row.chapter.number + " " + row.chapter.title }}
                  </div>
                </q-btn>
              </td>
              <td class="text-left">
                <q-btn
                  dense
                  color="white"
                  text-color="black"
                  :label="row.scan.name"
                  flat
                />
              </td>
            </tr>
          </tbody>
        </q-markup-table>
        <div class="row flex flex-center">
          <q-pagination v-model="currentPage" :max="maxPage" input />
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";
import { date } from "quasar";

export default defineComponent({
  name: "PageIndex",
  data() {
    return {
      loading: true,
      offset: 0,
      limit: 50,
      currentPage: 1,
      maxPage: 1000,
      columns: ["Title", "Chapter", "Group"],
      rows: [],
      pagination: {
        page: 1,
        rowsPerPage: 0,
      },
    };
  },
  computed: {
    language: {
      get() {
        return this.$store.state.mangadex.language;
      },
    },
  },
  watch: {
    currentPage: function (newPage, oldPage) {
      if (oldPage == 1) {
        this.limit = 50;
      }
      this.offset = newPage * this.limit;
      this.flag();
      this.fetchChapterList();
    },
  },
  methods: {
    flag() {
      this.loading = !this.loading;
    },
    async fetchChapterList() {
      var manga = {};
      var scan = {};
      this.rows = [];
      await this.$api
        .get(
          "chapter?offset=" +
            this.offset +
            "&limit=" +
            this.limit +
            "&includes[]=manga" +
            "&includes[]=scanlation_group" +
            "&order[publishAt]=desc" +
            "&translatedLanguage[]=" +
            this.language
        )
        .then((response) => {
          for (var result of response.data.results) {
            var title = "Unknown";
            for (var relationship of result.relationships) {
              if (relationship.type == "manga") {
                manga = {
                  id: relationship.id,
                  title: relationship.attributes.title.en,
                };
              }
              if (relationship.type == "scanlation_group") {
                scan = {
                  id: relationship.id,
                  name: relationship.attributes.name,
                };
              }
            }
            if (result.data.attributes.title != "") {
              if (result.data.attributes.title != null) {
                title = result.data.attributes.title;
              }
            }
            this.rows.push({
              date: date.formatDate(
                result.data.attributes.publishAt,
                "YYYY-MM-DD HH:mm:ss"
              ),
              manga: manga,
              chapter: {
                id: result.data.id,
                number: result.data.attributes.chapter,
                title: title,
              },
              scan: scan,
            });
          }
          this.flag();
        })
        .catch(function (error) {
          if (error.response) {
            // Request made and server responded
            console.log(error.response.data);
            console.log(error.response.status);
            console.log(error.response.headers);
          } else if (error.request) {
            // The request was made but no response was received
            console.log(error.request);
          } else {
            // Something happened in setting up the request that triggered an Error
            console.log("Error", error.message);
          }
        });
    },
  },
  async mounted() {
    await this.fetchChapterList();
  },
});
</script>
