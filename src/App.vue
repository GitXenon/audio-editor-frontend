<script setup lang="ts">
import { computed, ref, onMounted } from "vue";
import { Container, Draggable } from "vue-dndrop";
import axios from "axios";
import { applyDrag } from "@/utils";
import type { DropResult } from "@/types/Draggable";
import PlayButton from "@/components/PlayButton.vue";
import RecordButton from "@/components/RecordButton.vue";
import UploadButton from "@/components/UploadButton.vue";

defineProps({
  val: {
    type: Number,
    default: 0,
  },
});

interface Audio {
  audio: HTMLAudioElement;
  name: string;
  color: string;
}

interface AudioR2 {
  audio: string;
  name: string;
  color: string;
}

let audiosR2: AudioR2[] = [];
const audios = ref<Audio[]>([]);
const currentTrackIndex = ref(0);
const isPlaying = ref(false);
const colorArray = [
  "#F94144",
  "#F3722C",
  "#F48C06",
  "#F9C74F",
  "#90BE6D",
  "#43AA8B",
  "#4D908E",
  "#577590",
  "#277DA1",
  "#7B2CBF",
];
const roomInput = ref("cloudflare-test");

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

function currentID(): string {
  return window.location.pathname.slice(1);
}

function idIsPresent(): boolean {
  let match = window.location.pathname.match("/.+");
  if (match !== null) {
    return match.length !== 0;
  } else {
    return false;
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
      let htmlAudioElement = await new Audio(reader.result);
      htmlAudioElement.addEventListener("ended", nextTrack);
      let randomlyAssignedColor = randomColor();
      audios.value.push({
        audio: htmlAudioElement,
        name: file.name,
        color: randomlyAssignedColor,
      });
      audiosR2.push({
        audio: reader.result,
        name: file.name,
        color: randomlyAssignedColor,
      });
      postAudioToWorker(JSON.stringify(audiosR2));
    }
  };
  reader.readAsDataURL(file);
}

function loadAudioFromWorker() {
  axios
    .get("https://api.xbuss.ca/audios/" + currentID(), {
      headers: { "content-type": "application/json" },
    })
    .then(function (response: any) {
      audiosR2 = response.data;
      audios.value = [];
      for (let i = 0; i < audiosR2.length; i++) {
        let htmlAudioElement = new Audio(audiosR2[i].audio);
        htmlAudioElement.addEventListener("ended", nextTrack);
        audios.value.push({
          audio: htmlAudioElement,
          name: audiosR2[i].name,
          color: audiosR2[i].color,
        });
      }
    });
}

function postAudioToWorker(source: string) {
  axios.post("https://api.xbuss.ca/audios/" + currentID(), source, {
    headers: { "content-type": "application/json" },
  });
}

function onDrop(dropResult: DropResult) {
  currentMusic.value?.pause();
  isPlaying.value = false;
  audios.value = applyDrag(audios.value, dropResult);
  audiosR2 = applyDrag(audiosR2, dropResult);
  postAudioToWorker(JSON.stringify(audiosR2));
  resetAllTracksToZero();
}

function resetAllTracksToZero() {
  for (let i = 0; i < audios.value.length; i++) {
    audios.value[i].audio.fastSeek(0);
  }
}

function randomColor() {
  return colorArray[Math.floor(Math.random() * colorArray.length)];
}

function roomTeleportation() {
  window.location.pathname = roomInput.value;
}

onMounted(() => {
  if (idIsPresent()) {
    loadAudioFromWorker();
  }
});

const currentMusic = computed(() => {
  if (audios.value.length !== 0) {
    return audios.value[currentTrackIndex.value].audio;
  } else {
    return null;
  }
});
</script>

<template>
  <div v-if="idIsPresent()">
    <div class="flex gap-1 place-content-start">
      <PlayButton id="play-button" @click="handlePlayButton">
        {{ isPlaying ? "Pause" : "Play" }}
      </PlayButton>
      <PlayButton id="get-button" @click="loadAudioFromWorker">
        Fetch audio
      </PlayButton>
      <RecordButton
        id="record-button"
        title="Does not work right now"
        @click="recordAudio"
      >
        Record
      </RecordButton>
      <UploadButton @change="addTrackFromFile" />
    </div>
    <div class="flex flex-row justify-items-start justify-center gap-y-5 p-2">
      <Container @drop="onDrop" orientation="horizontal" behaviour="contain">
        <Draggable v-for="audio in audios" :key="audio.audio.src">
          <div
            class="draggable-item-horizontal cursor-move rounded-md h-24 w-48"
            :style="{ backgroundColor: audio.color }"
          >
            <h4 class="text-xs font-mono text-white">{{ audio.name }}</h4>
          </div>
        </Draggable>
      </Container>
    </div>
  </div>
  <div class="grid place-content-center" v-else>
    <p class="py-3 text-base font-normal text-gray-700">
      Collaborate online across multiple devices on a shared space. To create a
      new room or join an existing room, enter a room identifier and click 'GO'.
    </p>
    <div class="flex flex-auto place-content-center">
      <input
        v-model="roomInput"
        class="form-control px-3 py-1.5 text-base font-normal text-gray-700 bg-white bg-clip-padding border border-solid border-gray-300 rounded-l-lg transition ease-in-out m-0 focus:text-gray-700 focus:bg-white focus:border-blue-600 focus:outline-none"
      />
      <button
        class="rounded-r-lg bg-blue-200 px-3 py-1.5 text-gray-700 tet-base font-normal bg-clip-padding"
        @click="roomTeleportation()"
      >
        Go
      </button>
    </div>
  </div>
</template>
