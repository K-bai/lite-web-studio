<script>
import { defineComponent, ref, onMounted } from "vue";
export default defineComponent({
  name: "SongFilterMyCollection",
});
</script>

<script setup>
import utils from "utils/utils.js";
import bus from "vue3-eventbus";

const props = defineProps({
  songListFiltered: Array,
});

onMounted(() => {
  bus.on("refresh-collection", () => {
    songMyCollection.value = [...utils.readMyCollection()];
  });
});


const songMyCollection = ref(window.AudioLists.song_my_collection);
const isDeletingCollection = ref(false);
const addCollectionName = ref("");

const replaceCollection = (songList) => {
  bus.emit(
    "playlist-replace-event",
    songList.filter((s) => s.has_audio)
  );
  //this.$parent.$parent.$refs.player.playlist_replace();
};

const clickCollection = (idx) => {
  if (isDeletingCollection.value) {
    songMyCollection.value.splice(idx, 1);
    isDeletingCollection.value = false;
  }
  else {
    replaceCollection(songMyCollection.value[idx].list);
  }
};

const addPressEnter = (event) => {
  addCollection();
  event.target.blur();
};

const addCollection = () => {
  // 没写名字就不管
  if (addCollectionName.value === '') return;
  // 歌单是空的就不管
  if (window.AudioLists.playlist[0].id === 'empty_song') return;

  // 查找同名歌单
  let idx = songMyCollection.value.map(c=>c.name).findIndex(n=>(n === addCollectionName.value));
  if (idx !== -1) {
    // 已有同名就更新
    songMyCollection.value[idx].list = [...window.AudioLists.playlist];
  }
  else {
    // 否则新建
    songMyCollection.value.push({
      name: addCollectionName.value,
      list: [...window.AudioLists.playlist]
    });
  }
  // 写入localstorage
  utils.saveMyCollection(songMyCollection.value);
};

</script>

<template>
  <div>
    <div class="title">我的歌单</div>
    <div class="c-song-collection">
      <div
        v-bind:class="['collection-item', {'collection-item-deleting': isDeletingCollection}]"
        v-for="(collection, index) in songMyCollection"
        v-bind:key="collection.name"
        v-on:click="clickCollection(index)"
        v-bind:title="isDeletingCollection ? '删除该歌单' : '替换播放列表'"
      >
        <img src="node_modules/bootstrap-icons/icons/bookmark-star.svg?url" v-show="!isDeletingCollection" />
        <img src="node_modules/bootstrap-icons/icons/trash-fill.svg?url" v-show="isDeletingCollection" />
        <div>{{ collection.name }}</div>
      </div>
      <button
        class="general-button general-button-red control-item"
        v-on:click="isDeletingCollection = !isDeletingCollection"
        v-show="songMyCollection.length !== 0"
      >
        {{isDeletingCollection ? '取消' : '删除歌单'}}
      </button>
      <div class="c-add-collection control-item">
        <input
          v-model="addCollectionName"
          placeholder="歌单名"
          class="general-input add-collection-input"
          v-on:keydown.enter="addPressEnter"
          v-on:keydown.space.stop=""
        />
        <button
          class="general-button general-button-grey add-collection-button"
          v-on:click="addCollection()"
        >添加当前播放列表为歌单</button>
      </div>
    </div>
  </div>
</template>

<style scoped>

.title {
  margin: 0.5rem;
  font-size: 1.3rem;
}
.title-filter {
  display: flex;
  align-items: flex-end;
}
.title-filter-expand {
  font-size: 0.9rem;
  color: blueviolet;
  margin-left: 1rem;
  user-select: none;
  cursor: pointer;
}

.c-song-collection {
  margin-left: 1rem;
  margin-right: 1rem;
  margin-bottom: 1rem;
  display: flex;
  flex-wrap: wrap;
}
.collection-item {
  padding-top: 0.3rem;
  padding-bottom: 0.3rem;
  padding-left: 0.6rem;
  padding-right: 0.6rem;
  margin-top: 0.2rem;
  margin-bottom: 0.2rem;
  margin-right: 1rem;
  border-bottom: 1px rgb(190, 190, 190) solid;
  color: #2a2d30;
  user-select: none;
  cursor: pointer;
  display: flex;
  align-items: flex-end;
}
@media (any-hover: hover) {
  .collection-item:hover{
    background-color: rgba(0, 0, 0, 0.05);
  }
}
.collection-item:active {
  background-color: rgba(0, 0, 0, 0.1);
}
.collection-item div {
  margin-left: 0.3rem;
}
.collection-item-deleting {
  border-bottom: 2px rgb(200, 0, 0) solid;
}


.control-item {
  margin-right: 1rem;
  margin-top: 0.2rem;
  margin-bottom: 0.2rem;
  min-height: 2rem;
}
.c-add-collection {
  display: flex;
  align-items: stretch;
}
.add-collection-input {
  border-top-right-radius: 0px;
  border-bottom-right-radius: 0px;
  border-right: none;
}
.add-collection-button {
  border-top-left-radius: 0px;
  border-bottom-left-radius: 0px;
}

</style>
