<template>
  <q-page padding>
    <div>
      <q-breadcrumbs>
        <q-breadcrumbs-el label="Home" icon="home" to="/" />
        <q-breadcrumbs-el :label="manga.title" :to="'/manga/' + manga.id" />
        <q-breadcrumbs-el
          :label="'Chapter ' + chapter.number + ' ' + chapter.title"
        />
      </q-breadcrumbs>
    </div>

    <div class="row justify-center">
      <div class="col-6">
        <q-btn
          flat
          dense
          icon="navigate_before"
          class="full-width"
          :to="'/chapter/' + prev.id"
          v-if="prev.state"
        />
        <q-btn
          flat
          dense
          icon="navigate_before"
          class="full-width"
          v-else
          disabled
        />
      </div>
      <div class="col-6">
        <q-btn
          flat
          dense
          icon="navigate_next"
          class="full-width"
          :to="'/chapter/' + next.id"
          v-if="next.state"
        />
        <q-btn
          flat
          dense
          icon="navigate_next"
          class="full-width"
          v-else
          disabled
        />
      </div>
    </div>
    <div class="row justify-center" v-if="loading">
      <q-circular-progress
        indeterminate
        size="50px"
        color="primary"
        class="q-ma-md"
      />
    </div>
    <div class="row fit justify-center" v-else>
      <div
        class="row justify-center"
        style="max-width: 90%; width: 50vw"
        v-for="page in chapter.pagesUrl"
        :key="page"
      >
        <Chapter :src="page" />
      </div>
    </div>
    <div class="row justify-center">
      <div class="col-6">
        <q-btn
          flat
          dense
          icon="navigate_before"
          class="full-width"
          v-if="prev.state"
        />
        <q-btn
          flat
          dense
          icon="navigate_before"
          class="full-width"
          v-else
          disabled
        />
      </div>
      <div class="col-6">
        <q-btn
          flat
          dense
          icon="navigate_next"
          class="full-width"
          v-if="next.state"
        />
        <q-btn
          flat
          dense
          icon="navigate_next"
          class="full-width"
          v-else
          disabled
        />
      </div>
    </div>
  </q-page>
</template>

<script>
import Chapter from "src/components/Chapter.vue";
export default {
  // name: 'PageName',
  components: { Chapter },
  data() {
    return {
      manga: {
        id: "",
        title: "",
      },
      scan: {
        id: "",
        name: "",
      },
      next: {
        id: "",
        state: false,
        number: "",
      },
      prev: {
        id: "",
        state: false,
        number: "",
      },
      pageImage: {},
      baseUrl: "",
      chapter: {
        number: "",
        title: "",
        pages: [],
        volume: null,
        pagesUrl: [],
      },
      volumes: [],
      loading: true,
    };
  },
  methods: {
    async fetchAggregate() {
      var volumes = {};
      let chapter_url = "";
      await this.$api
        .get("manga/" + this.manga.id + "/aggregate?translatedLanguage[]=en", {
          headers: {
            "Content-Type": "application/json",
          },
        })
        .then((response) => {
          //   console.log(response.data);
          var sort = [];
          let volume_name = "";
          let chapter_url = "";
          for (var volume in response.data.volumes) {
            if (volume == "none") {
              volume_name = "Unknown";
            } else {
              volume_name = volume;
            }
            for (var chapter in response.data.volumes[volume].chapters) {
              sort[sort.length] = Number(chapter);
            }
          }
          sort.sort(function (a, b) {
            return a - b;
          });
          // console.log(sort);
          for (var item of sort) {
            chapter_url = chapter_url + "&chapter[]=" + item;
          }
          volumes[volume_name] = {
            chapter_url: chapter_url,
            volume_name: volume_name,
          };
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
      for (var volume in volumes) {
        // console.log(volume);
        // console.log(this.volumes[volume]);
        await this.fetchChapterList(volumes[volume]);
      }
      // console.log(this.chapter.volume);
      // console.log(this.volumes);
      for (var volume in this.volumes) {
        if (volume.volume_name == this.chapter.volume) {
          // console.log(this.volumes[volume].chapter_data);
          let ch = [];
          for (var chapters in this.volumes[volume].chapter_data) {
            ch.push(chapters);
          }
          // console.log(ch);
          // console.log(ch.indexOf(this.chapter.number));
          if (ch.indexOf(this.chapter.number) == -1) {
          } else {
            this.next.state = true;
            this.next.number = ch[ch.indexOf(this.chapter.number) + 1];
            chapter_url = chapter_url + "&chapter[]=" + this.next.number;
          }
          if (ch.indexOf(this.chapter.number) == 0) {
          } else {
            this.prev.state = true;
            this.prev.number = ch[ch.indexOf(this.chapter.number) - 1];
            chapter_url = chapter_url + "&chapter[]=" + this.prev.number;
          }
        } else {
          // console.log("next2");
        }
      }

      this.loading = false;
      await this.fetchChapterPagination(chapter_url);
    },
    async fetchChapterList(volume) {
      await this.$api
        .get(
          "chapter?manga=" +
            this.manga.id +
            volume.chapter_url +
            "&includes[]=scanlation_group" +
            "&includes[]=user" +
            "&limit=100&translatedLanguage[]=" +
            this.$store.state.mangadex.language
        )
        .then((response) => {
          // console.log(response.data.results);
          let chapter_data = {};
          let scan = {};
          let user = {};
          let title = "Unknown";

          for (var result of response.data.results) {
            // console.log(result);
            for (var relationship of result.relationships) {
              if (relationship.type == "scanlation_group") {
                scan = {
                  id: relationship.id,
                  name: relationship.attributes.name,
                };
              } else if (relationship.type == "user") {
                user = {
                  id: relationship.id,
                  username: relationship.attributes.username,
                };
              }
            }
            if (result.data.attributes.title != "") {
              title = result.data.attributes.title;
            }
            // chapter_data.push({
            //   id: result.data.id,
            //   number: result.data.attributes.chapter,
            //   title: title,
            //   publishAt: date.formatDate(
            //     result.data.attributes.publishAt,
            //     "YYYY-MM-DD"
            //   ),
            //   scan: scan,
            //   user: user,
            // });
            if (result.data.attributes.chapter in chapter_data) {
              chapter_data[result.data.attributes.chapter].push({
                id: result.data.id,
                title: title,
              });
            } else {
              chapter_data[result.data.attributes.chapter] = [];
              chapter_data[result.data.attributes.chapter].push({
                id: result.data.id,
                title: title,
              });
            }
          }
          // console.log(chapter_data);
          // this.volumes[volume.volume_name] = chapter_data;
          this.volumes.push({
            volume_name: volume.volume_name,
            chapter_data: chapter_data,
          });
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
    async fetchChapterPagination(chapter_url) {
      await this.$api
        .get("chapter?manga=" + this.manga.id + chapter_url)
        .then(
          (response) => {
            // console.log(response.data.results);
            for (var chapter of response.data.results) {
              // console.log(chapter.data);
              if (chapter.data.attributes.chapter == this.next.number) {
                this.next.id = chapter.data.id;
              }
              if (chapter.data.attributes.chapter == this.prev.number) {
                this.prev.id = chapter.data.id;
              }
            }
          }
          // console.log(this.volumes);
        )
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
    async fetchMangaInformation() {
      await this.$api
        .get("manga/" + this.manga.id + "/aggregate")
        .then((response) => {
          // console.log(response.data);
          for (var volume in response.data.volumes) {
            let volume_name = "";
            if (volume == "none") {
              volume_name = "Unknown";
            } else {
              // console.log(volume);
              volume_name = volume;
            }
            var chapters = [];
            console.log(response.data.volumes[volume].chapters);
          }
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
    async fetchChapterInformation() {
      await this.$api
        .get(
          "chapter/" +
            this.$route.params.chapterId +
            "?includes[]=scanlation_group" +
            "&includes[]=manga" +
            "&limit=100&translatedLanguage[]=" +
            this.$store.state.mangadex.language
        )
        .then(
          (response) => {
            this.chapter = {
              number: response.data.data.attributes.chapter,
              title: response.data.data.attributes.title,
              pages: response.data.data.attributes.data,
              hash: response.data.data.attributes.hash,
              volume: response.data.data.attributes.volume,
              pagesUrl: [],
            };
            for (var relationship of response.data.relationships) {
              if (relationship.type == "manga") {
                this.manga = {
                  id: relationship.id,
                  title: relationship.attributes.title.en,
                };
              } else if (relationship.type == "scanlation_group") {
                this.scan = {};
              }
            }
          }
          // console.log(this.volumes);
        )
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
    async fetchBaseUrl() {
      let baseUrl = "";
      await this.$api
        .get("at-home/server/" + this.$route.params.chapterId)
        .then((response) => {
          // console.log(response.data);
          baseUrl = response.data.baseUrl + "/data/" + this.chapter.hash;
          let pagesUrl = [];
          for (var page of this.chapter.pages) {
            pagesUrl.push(baseUrl + "/" + page);
          }
          this.chapter.pagesUrl = pagesUrl;
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
    await this.fetchChapterInformation();
    await this.fetchAggregate();
    await this.fetchBaseUrl();
  },
};
</script>
