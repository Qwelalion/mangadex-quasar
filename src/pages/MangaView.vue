<template>
  <q-page padding>
    <div class="row">
      <q-breadcrumbs>
        <q-breadcrumbs-el label="Home" icon="home" to="/" />
        <q-breadcrumbs-el :label="title" />
      </q-breadcrumbs>
    </div>
    <div class="row">
      <div class="col-4">
        <div class="row justify-center">
          <q-img
            :src="coverUrl"
            spinner-color="white"
            style="height: 100%; max-width: 20vw"
          />
        </div>
        <div class="row justify-center">
          <div class="row text-h5" v-if="loading">
            <q-skeleton type="text" width="25vw" />
          </div>
          <div v-else>
            <div class="row text-center text-h5">
              {{ title }}
            </div>
            <div class="row justify-center text-h5">
              <q-chip>
                <q-avatar :color="statusColor" text-color="white" />
                {{ status }}
              </q-chip>
            </div>
            <q-separator />
            <div class="row">
              <div class="col">
                <div class="row justify-center text-h7">Author</div>
                <div class="row justify-center">
                  <q-chip>
                    {{ author }}
                  </q-chip>
                </div>
              </div>
              <div class="col">
                <div class="row justify-center text-h7">Artist</div>
                <div class="row justify-center">
                  <q-chip>
                    {{ artist }}
                  </q-chip>
                </div>
              </div>
            </div>
            <q-separator />
            <div class="row">
              <div class="col">
                <div class="row justify-center text-h7">Theme</div>
                <div
                  class="row justify-center"
                  v-for="theme in themes"
                  :key="theme.id"
                >
                  <q-chip>
                    {{ theme.name }}
                  </q-chip>
                </div>
              </div>
              <q-separator />
              <div class="col">
                <div class="row justify-center text-h7">Genre</div>
                <div
                  class="row justify-center"
                  v-for="genre in genres"
                  :key="genre.id"
                >
                  <q-chip>
                    {{ genre.name }}
                  </q-chip>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="col">
        <div class="row justify-center vertical-center" v-if="loading">
          <q-spinner height="50vh" color="primary" size="3em" :thickness="10" />
        </div>
        <div class="row" v-else style="width: 90%">
          <div class="row justify-center" style="width: 90%">
            <q-tabs
              v-model="tab"
              dense
              class="text-grey"
              active-color="primary"
              indicator-color="primary"
              align="justify"
              narrow-indicator
              outside-arrows
              mobile-arrows
              style="max-width: 50vw"
            >
              <q-tab name="description" label="Description" />
              <q-tab name="chapter" label="Chapters" />
              <q-tab name="review" label="Reviews" disable />
            </q-tabs>
          </div>
          <q-separator />

          <div class="row justify-center" style="width: 90%">
            <q-tab-panels v-model="tab" keep-alive>
              <q-tab-panel name="description">
                <div class="text-center text-h6">Sypnosis</div>
                <div class="text-center">
                  {{ description }}
                </div>
              </q-tab-panel>

              <q-tab-panel name="ChaptersList">
                <router-view v-slot="{ ChaptersList }">
                  <keep-alive>
                    <component :is="ChaptersList" />
                  </keep-alive>
                </router-view>
              </q-tab-panel>

              <q-tab-panel name="chapter">
                <ChaptersList :mangaId="$route.params.mangaId" />
              </q-tab-panel>

              <q-tab-panel name="review">
                <div class="text-h6">Movies</div>
                Lorem ipsum dolor sit amet consectetur adipisicing elit.
              </q-tab-panel>
            </q-tab-panels>
          </div>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script>
import ChaptersList from "src/components/ChapterList.vue";
export default {
  // name: 'PageName',
  components: { ChaptersList },
  data() {
    return {
      title: "",
      author: "",
      artist: "",
      coverUrl: "",
      status: "",
      description: "",
      themes: [],
      genres: [],
      tab: "description",
      loading: true,
    };
  },
  computed: {
    statusColor() {
      return status === "ongoing" ? "blue" : "green";
    },
  },
  methods: {
    async fetchMangaInformation() {
      await this.$api
        .get(
          "manga/" +
            this.$route.params.mangaId +
            "?includes[]=author" +
            "&includes[]=artist" +
            "&includes[]=cover_art"
        )
        .then((response) => {
          // console.log(response.data);
          this.title = response.data.data.attributes.title.en;
          this.status = response.data.data.attributes.status;
          this.description = response.data.data.attributes.description.en;
          for (var tag of response.data.data.attributes.tags) {
            if (tag.attributes.group == "theme") {
              this.themes.push({ name: tag.attributes.name.en, id: tag.id });
            } else if (tag.attributes.group == "genre") {
              this.genres.push({ name: tag.attributes.name.en, id: tag.id });
            }
          }
          for (var relationship of response.data.relationships) {
            if (relationship.type == "author") {
              if (relationship.attributes.name == "") {
                this.author = "N/A";
              } else {
                this.author = relationship.attributes.name;
              }
            }
            if (relationship.type == "artist") {
              if (relationship.attributes.name == "") {
                this.artist = "N/A";
              } else {
                this.artist = relationship.attributes.name;
              }
            } else if (relationship.type == "cover_art") {
              this.coverUrl =
                "https://uploads.mangadex.org/covers/" +
                this.$route.params.mangaId +
                "/" +
                relationship.attributes.fileName;
              // + ".256.jpg";
            }
          }
          this.loading = false;
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
    await this.fetchMangaInformation();
  },
};
</script>
