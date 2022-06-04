<script setup lang="ts">
import { computed, ref } from "vue";
import { Container, Draggable } from "vue-dndrop";
import { applyDrag } from "@/utils";
import type { DropResult } from "@/types/Draggable";
import PlayButton from "@/components/PlayButton.vue"
import RecordButton from "@/components/RecordButton.vue";
import UploadButton from "@/components/UploadButton.vue";

defineProps({
  val: {
    type: Number,
    default: 0,
  },
});

const audios = ref<Array<{ audio: HTMLAudioElement; name: string, color: string}>>([]);
const currentTrackIndex = ref(0);
const isPlaying = ref(false);
const colorArray = ["#F94144",
  "#F3722C",
  "#F48C06",
  "#F9C74F",
  "#90BE6D",
  "#43AA8B",
  "#4D908E",
  "#577590",
  "#277DA1",
  "#7B2CBF",
]

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
  if (currentMusic.value != null) {
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
  const file = event.target.files[0];
  const reader = new FileReader();
  reader.onloadend = async function () {
    if (typeof reader.result === "string") {
      let htmlAudioElement = await new Audio(reader.result)
      htmlAudioElement.addEventListener("ended", nextTrack)
      audios.value.push({ audio: htmlAudioElement, name: file.name, color: randomColor() });
    }
  };
  reader.readAsDataURL(file);
}

function onDrop(dropResult: DropResult) {
  currentMusic.value?.pause();
  isPlaying.value = false;
  audios.value = applyDrag(audios.value, dropResult);
  resetAllTracksToZero()
}

function resetAllTracksToZero() {
  for (let i = 0; i < audios.value.length; i++) {
    audios.value[i].audio.fastSeek(0)
  }
}

function randomColor() {
  return colorArray[Math.floor(Math.random()*colorArray.length)]
}

const currentMusic = computed(() => {
  if (audios.value.length !== 0) {
    return audios.value[currentTrackIndex.value].audio;
  } else {
    return null;
  }
});
</script>

<template>
  <div class="flex gap-1 place-content-start">
    <PlayButton id="play-button" @click="handlePlayButton">
      {{ isPlaying ? "Pause" : "Play" }}
    </PlayButton>
    <RecordButton id="record-button" @click="recordAudio">
      Record
    </RecordButton>
    <UploadButton @change="addTrackFromFile" />
  </div>
  <div class="flex flex-row justify-items-start justify-center gap-y-5 p-2">
    <Container @drop="onDrop" orientation="horizontal" behaviour="contain">
      <Draggable v-for="audio in audios">
        <div class="draggable-item-horizontal cursor-move rounded-md h-24 w-48"  :style="{backgroundColor: audio.color}">
          <h4 class="text-xs font-mono text-white">{{ audio.name }}</h4>
        </div>
      </Draggable>
    </Container>
  </div>
</template>
