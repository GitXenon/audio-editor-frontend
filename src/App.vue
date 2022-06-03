<script setup lang="ts">
import { computed, ref } from "vue";
import { Container, Draggable } from "vue-dndrop";
import { applyDrag } from "./utils";
import type { DropResult } from "./types/Draggable";

defineProps({
  val: {
    type: Number,
    default: 0,
  },
});

const audios = ref<Array<{ src: string; name: string }>>([]);
const currentTrackIndex = ref(0);
const isPlaying = ref(false);
const audioList = ref<Array<HTMLAudioElement>>([]);

async function playMusic() {
  if (currentMusic.value !== null) {
    try {
      await currentMusic.value.play();
      isPlaying.value = true;
    } catch (err) {
      isPlaying.value = false;
    }
  }
}

function handlePlayButton() {
  if (currentMusic.value !== null) {
    if (currentMusic.value.paused) {
      playMusic();
    } else {
      currentMusic.value.pause();
      isPlaying.value = false;
    }
  }
}

function nextTrack() {
  if (audios.value.length > currentTrackIndex.value + 1) {
    currentTrackIndex.value++;
    playMusic();
  } else {
    isPlaying.value = false;
    currentTrackIndex.value = 0;
  }
}

function recordAudio() {
  // TODO: Add this feature
  //if (navigator.mediaDevices.getUserMedia) {
  //const constraints = { audio: true };
  //let onSuccess = function (stream: any) {};
  // let onError = function (e: any) {};
  // navigator.mediaDevices.getUserMedia(constraints).then(onSuccess, onError);
  //}
}

function addTrackFromFile(event: any) {
  console.log(event);
  console.log(event.type);
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onloadend = function () {
    if (typeof reader.result === "string") {
      audios.value.push({ src: reader.result, name: file.name });
    }
  };
  reader.readAsDataURL(file);
}

function onDrop(dropResult: DropResult) {
  audios.value = applyDrag(audios.value, dropResult);
}

const currentMusic = computed(() => {
  if (audioList.value.length !== 0) {
    return audioList.value[currentTrackIndex.value];
  } else {
    return null;
  }
});
</script>

<template>
  <h1>Audio App v.0.1</h1>
  <button id="play-button" @click="handlePlayButton">
    {{ isPlaying ? "Pause" : "Play" }}
  </button>
  <button id="record-button" class="record-button" @click="recordAudio">
    Record
  </button>
  <input type="file" @change="addTrackFromFile" />
  <div style="overflow-x: auto">
    <Container
      @drop="onDrop"
      class="box"
      orientation="horizontal"
      behaviour="contain"
    >
      <Draggable v-for="audio in audios" class="audio-card" v-bind:key="audio">
        <div class="draggable-item-horizontal">
          <audio :src="audio.src" @ended="nextTrack" ref="audioList"></audio>
          <h4>{{ audio.name }}</h4>
        </div>
      </Draggable>
    </Container>
  </div>
</template>

<style>
body {
  font-family: Roboto, sans-serif;
}

.audio-card {
  background-color: whitesmoke;
  flex: 1;
  text-align: center;
}

.record-button {
  background-color: red;
  color: white;
}

.draggable-item-horizontal {
  padding: 1rem;
}

.box {
  border: solid;
  display: flex;
}
</style>
