<template>
  <q-page class="flex flex-center">
    <div class="q-ma-md" style="width: 90%; height: 100%">
      <div>
        <q-form @submit="onSubmit" @reset="onReset" class="q-gutter-md">
          <q-input filled v-model="manga.title" label="Manga Title" />

          <q-input filled v-model="manga.author" label="Author" />
          <q-expansion-item expand-separator label="Include Tags">
            <q-card>
              <q-card-section> Format </q-card-section>
              <q-card-section>
                <div class="col">
                  <q-checkbox
                    v-for="format in tags.format"
                    :key="format.id"
                    v-model="manga.tags"
                    :val="format.id"
                    :label="format.name"
                  />
                </div>
              </q-card-section>
            </q-card>
            <q-card>
              <q-card-section> Genre </q-card-section>
              <q-card-section>
                <div class="col">
                  <q-checkbox
                    v-for="genre in tags.genre"
                    :key="genre.id"
                    v-model="manga.tags"
                    :val="genre.id"
                    :label="genre.name"
                  />
                </div>
              </q-card-section>
            </q-card>
            <q-card>
              <q-card-section> Theme </q-card-section>
              <q-card-section>
                <div class="col">
                  <q-checkbox
                    v-for="theme in tags.theme"
                    :key="theme.id"
                    v-model="manga.tags"
                    :val="theme.id"
                    :label="theme.name"
                  />
                </div>
              </q-card-section>
            </q-card>
          </q-expansion-item>
          <q-expansion-item expand-separator label="Exclude Tags">
            <q-card>
              <q-card-section> Format </q-card-section>
              <q-card-section>
                <div class="col">
                  <q-checkbox
                    v-for="format in tags.format"
                    :key="format.id"
                    v-model="manga.exclude"
                    :val="format.id"
                    :label="format.name"
                  />
                </div>
              </q-card-section>
            </q-card>
            <q-card>
              <q-card-section> Genre </q-card-section>
              <q-card-section>
                <div class="col">
                  <q-checkbox
                    v-for="genre in tags.genre"
                    :key="genre.id"
                    v-model="manga.exclude"
                    :val="genre.id"
                    :label="genre.name"
                  />
                </div>
              </q-card-section>
            </q-card>
            <q-card>
              <q-card-section> Theme </q-card-section>
              <q-card-section>
                <div class="col">
                  <q-checkbox
                    v-for="theme in tags.theme"
                    :key="theme.id"
                    v-model="manga.exclude"
                    :val="theme.id"
                    :label="theme.name"
                  />
                </div>
              </q-card-section>
            </q-card>
          </q-expansion-item>

          <div class="row justify-center">
            <q-btn label="Submit" type="submit" color="primary" />
            <q-btn
              label="Reset"
              type="reset"
              color="primary"
              flat
              class="q-ml-sm"
            />
          </div>
        </q-form>
      </div>
      <div v-if="searchFlag"></div>
      <div v-else>
        <div class="row justify-center vertical-center" v-if="searchLoading">
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
                <td>
                  <q-btn
                    dense
                    color="white"
                    text-color="black"
                    :label="row.manga.title"
                    :to="'/manga/' + row.manga.id"
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
    </div>
  </q-page>
</template>

<script>
import { defineComponent } from "vue";

export default defineComponent({
  name: "PageSearch",
  data() {
    return {
      loading: true,
      searchFlag: true,
      searchLoading: true,
      offset: 0,
      limit: 50,
      currentPage: 1,
      maxPage: 1000,
      manga: {
        title: "",
        author: "",
        tags: [],
        exclude: [],
      },
      tags: {
        format: [],
        theme: [],
        genre: [],
        content: [],
      },
      columns: ["Title"],
      rows: [],
      pagination: {
        page: 1,
        rowsPerPage: 0,
      },
    };
  },
  computed: {},
  watch: {
    currentPage: function (newPage, oldPage) {
      if (oldPage == 1) {
        this.limit = 50;
      }
      this.offset = newPage * this.limit;
      this.flag();
      this.searchManga();
    },
  },
  methods: {
    flag() {
      this.searchLoading = !this.searchLoading;
    },
    async onSubmit() {
      this.searchFlag = false;
      this.currentPage = 1;
      this.searchManga();
    },

    onReset() {
      this.manga = {
        title: "",
        author: "",
        tags: [],
      };
    },
    async searchManga() {
      let tags = "";
      let exctags = "";
      var manga = {};
      this.rows = [];
      if (this.manga.tags.length > 0) {
        for (var tag of this.manga.tags) {
          tags = tags + "&includedTags[]=" + tag;
        }
      }
      if (this.manga.tags.exclude > 0) {
        for (var tag of this.manga.tags) {
          exctags = exctags + "&excludedTags[]=" + tag;
        }
      }
      await this.$api
        .get(
          "manga?title=" +
            this.manga.title +
            tags +
            exctags +
            "&limit=" +
            this.limit +
            "&offset=" +
            this.offset +
            "&order[updatedAt]=desc"
        )
        .then((response) => {
          for (var result of response.data.results) {
            manga = {
              id: result.data.id,
              title: result.data.attributes.title.en,
            };
            this.rows.push({
              manga: manga,
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
    async fetchTagsList() {
      this.rows = [];
      await this.$api
        .get("manga/tag")
        .then((response) => {
          for (var result of response.data) {
            if (result.data.attributes.group == "format") {
              this.tags.format.push({
                name: result.data.attributes.name.en,
                id: result.data.id,
              });
            } else if (result.data.attributes.group == "theme") {
              this.tags.theme.push({
                name: result.data.attributes.name.en,
                id: result.data.id,
              });
            } else if (result.data.attributes.group == "genre") {
              this.tags.genre.push({
                name: result.data.attributes.name.en,
                id: result.data.id,
              });
            } else if (result.data.attributes.group == "content") {
              this.tags.content.push({
                name: result.data.attributes.name.en,
                id: result.data.id,
              });
            }
          }
          //   console.log(this.tags);
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
    await this.fetchTagsList();
  },
});
</script>
