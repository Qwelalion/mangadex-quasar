<template>
  <div>
    <div v-if="loading">
      <q-circular-progress
        indeterminate
        size="50px"
        color="primary"
        class="q-ma-md"
      />
    </div>
    <div v-else>
      <div
        class="row justify-center"
        v-for="volume of volumes"
        :key="volume.volume_name"
      >
        <q-expansion-item
          expand-separator
          :label="'Volume ' + volume.volume_name"
        >
          <q-list bordered separator>
            <q-item
              v-for="(chapter, number) in volume.chapter_data"
              :key="chapter"
            >
              <q-item-section>
                <q-item-label> Chapter: {{ number }} </q-item-label>
                <q-item v-for="chapter_info of chapter" :key="chapter_info.id">
                  <q-item-section>
                    <q-item-label caption>
                      <q-btn
                        class="gt-xs"
                        size="12px"
                        flat
                        dense
                        :label="chapter_info.title"
                        :to="{
                          name: 'chapter',
                          params: {
                            chapterId: chapter_info.id,
                          },
                        }"
                        custom
                      />
                    </q-item-label>
                    <q-item-label caption>
                      <q-icon name="groups" />{{ chapter_info.scan.name }}
                    </q-item-label>
                  </q-item-section>

                  <q-item-section side top>
                    <q-item-label caption>
                      <q-icon name="schedule" />{{ chapter_info.publishAt }}
                    </q-item-label>
                    <q-item-label caption>
                      <q-icon name="person" />{{ chapter_info.user.username }}
                    </q-item-label>
                  </q-item-section>
                </q-item>
              </q-item-section>
            </q-item>
          </q-list>
        </q-expansion-item>
      </div>
    </div>
  </div>
</template>

<script>
import { date } from "quasar";
export default {
  name: "ChaptersList",
  props: {
    mangaId: {
      type: String,
      default: "",
    },
  },
  data() {
    return {
      loading: true,
      volumes: [],
    };
  },
  methods: {
    async fetchAggregate() {
      var volumes = {};
      await this.$api
        .get("manga/" + this.mangaId + "/aggregate?translatedLanguage[]=en", {
          headers: {
            "Content-Type": "application/json",
          },
        })
        .then((response) => {
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
        await this.fetchChapterList(volumes[volume]);
      }
      this.loading = false;
    },
    async fetchChapterList(volume) {
      await this.$api
        .get(
          "chapter?manga=" +
            this.mangaId +
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

          for (var result of response.data.results) {
            // console.log(result);
            let title = "Unknown";
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
            if (result.data.attributes.chapter in chapter_data) {
              chapter_data[result.data.attributes.chapter].push({
                id: result.data.id,
                title: title,
                publishAt: date.formatDate(
                  result.data.attributes.publishAt,
                  "YYYY-MM-DD"
                ),
                scan: scan,
                user: user,
              });
            } else {
              chapter_data[result.data.attributes.chapter] = [];
              chapter_data[result.data.attributes.chapter].push({
                id: result.data.id,
                title: title,
                publishAt: date.formatDate(
                  result.data.attributes.publishAt,
                  "YYYY-MM-DD"
                ),
                scan: scan,
                user: user,
              });
            }
          }
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
  },
  async mounted() {
    await this.fetchAggregate();
  },
};
</script>
