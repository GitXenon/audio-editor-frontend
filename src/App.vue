<script setup lang="ts">
import {computed, ref} from 'vue'

defineProps({
  val: {
    type: Number,
    default: 0
  }
})

const audios = ref<Array<string>>([])
const currentTrackIndex = ref(0)
const isPlaying = ref(false)
const audioList = ref<Array<HTMLAudioElement>>([])

async function playMusic() {
  if (currentMusic.value !== null) {
    try {
      await currentMusic.value.play()
      isPlaying.value = true
    } catch (err) {
      isPlaying.value = false
    }
  }
}

function handlePlayButton() {
  if (currentMusic.value !== null) {
    console.log(currentMusic.value)
    if (currentMusic.value.paused) {
      playMusic()
    } else {
      currentMusic.value.pause()
      isPlaying.value = false
    }
  }
}

function nextTrack() {
  if (audios.value.length > currentTrackIndex.value + 1) {
    currentTrackIndex.value++
    playMusic()
  } else {
    isPlaying.value = false
    currentTrackIndex.value = 0
  }
}

function recordAudio() {
  if (navigator.mediaDevices.getUserMedia) {
    const constraints = { audio: true };

    let onSuccess = function(stream: any) {

    }

    let onError = function(e: any) {

    }

    navigator.mediaDevices.getUserMedia(constraints).then(onSuccess, onError);
  }
}

function visualize() {
}

function addTrackFromFile(event : any) {
  const file = event.target.files[0]
  const reader = new FileReader()
  reader.onloadend = function() {
    if (typeof reader.result === "string") {
      audios.value.push(reader.result)
    }
  }
  reader.readAsDataURL(file)
}

const currentMusic = computed(() => {
  if (audioList.value.length !== 0) {
    return audioList.value[currentTrackIndex.value]
  } else {
    return null
  }
})
</script>

<template>
  <h1>Audio App v.0.1</h1>
  <button id="play-button" @click="handlePlayButton">{{ isPlaying ? "Pause" : "Play" }}</button>
  <button id="upload-button">Upload</button>
  <button id="record-button" class="record-button" @click="recordAudio">Record</button>
  <input type="file" @change="addTrackFromFile">
  <div class="box">
    <div v-for="(item, index) in audios" class="audio-card">
      <audio :src="item" @ended="nextTrack" @loadeddata="visualize" ref="audioList"></audio>
      <h4>{{ index }}</h4>
    </div>
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
  color: white
}

.box {
  border: solid;
  display: flex;
}

</style>