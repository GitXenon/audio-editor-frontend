<script setup lang="ts">
import {computed, ref} from 'vue'

defineProps({
  val: {
    type: Number,
    default: 0
  }
})

const audios = ref([{'src': 'https://cdn.freesound.org/previews/24/24930_113878-lq.mp3'}, {'src': 'https://cdn.freesound.org/previews/634/634804_839843-lq.mp3'}, {'src': 'https://cdn.freesound.org/previews/635/635073_9177297-lq.mp3'}])
const currentTrackIndex = ref(0)
const isPlaying = ref(false)
const audioCtx = new AudioContext()

async function playMusic() {
  try {
    await currentMusic.value.play()
    isPlaying.value = true
  } catch (err) {
    isPlaying.value = false
  }
}

function handlePlayButton() {
  if (currentMusic.value.paused) {
    playMusic()
  } else {
    currentMusic.value.pause()
    isPlaying.value = false
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

function playSample(audioBuffer: AudioBuffer, time: number) {
  const sampleSource = audioCtx.createBufferSource()
  sampleSource.buffer = audioBuffer
  sampleSource.connect(audioCtx.destination)
  sampleSource.start(time)
  return sampleSource
}

const currentMusic = computed(() => {
  return audioList.value[currentTrackIndex.value]
})

const audioList = computed(() => {
  return document.querySelectorAll("audio")
})
</script>

<template>
  <h1>Audio App v.0.1</h1>
  <button id="play-button" @click="handlePlayButton">{{ isPlaying ? "Pause" : "Play" }}</button>
  <button id="upload-button">Upload</button>
  <button id="record-button" class="record-button" @click="recordAudio">Record</button>
  <div class="box">
    <div v-for="(item, index) in audios" class="audio-card">
      <audio :src="item.src" @ended="nextTrack" @loadeddata="visualize"></audio>
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