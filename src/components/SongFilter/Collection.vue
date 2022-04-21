<script>
import { defineComponent, ref, onMounted } from "vue";
export default defineComponent({
  name: "SongFilterCollection",
});
</script>

<script setup>
import bus from "vue3-eventbus";

const props = defineProps({
  songListFiltered: Array,
});


const songCollection = ref(window.AudioLists.song_collection);

const replaceCollection = (songList) => {
  bus.emit(
    "playlist-replace-event",
    songList.filter((s) => s.has_audio)
  );
  //this.$parent.$parent.$refs.player.playlist_replace();
};

onMounted(() => {
  bus.on("refresh-collection", () => {
    songCollection.value = [...window.AudioLists.song_collection];
  });
});

</script>

<template>
  <div>
    <div class="title">歌单</div>
    <div class="c-song-collection">
      <div
        class="collection-item"
        v-for="collection in songCollection"
        v-bind:key="collection.name"
        v-on:click="replaceCollection(collection.list)"
      >
        <img src="node_modules/bootstrap-icons/icons/tag.svg?url" />
        <div>{{ collection.name }}</div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import "styles/songfilter.scss";
</style>
