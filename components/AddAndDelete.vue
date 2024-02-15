<template>
  <button
    class="h-12 mx-1"
    @click="addDiv">
    新增 Div
  </button>

  <button
    class="h-12 mx-1"
    @click="openAddVideoModal">
    新增影片
  </button>

  <div
    v-if="showAddVideoModal"
    class="absolute top-0 left-0 w-full h-full flex items-center justify-center bg-black bg-opacity-50 z-10">
    <div className="bg-white p-4 rounded-lg shadow-lg text-center">
      <input
        v-model="videoUrl"
        placeholder="請輸入影片網址" />
      <button @click="addVideo">新增</button>
      <button @click="showAddVideoModal = false">取消</button>
    </div>
  </div>

  <div></div>

  <div class="flex">
    <div
      id="leftDiv"
      class="flex-2 max-w-[70%] bg-green-200 w-full h-[800px]">
      <div
        v-for="(div, index) in divs"
        :key="index"
        @mousedown="(event) => startDrag(event, index)"
        @dblclick="showAlert"
        :style="{ position: 'absolute', left: `${div.x}px`, top: `${div.y}px` }"
        class="w-80 h-80 bg-red-200">
        <template v-if="div.type === 'video'">
          <video
            :src="div.url"
            :id="`video-${index}`"></video>
        </template>
        <template v-else>
          {{ div.name }}
        </template>
      </div>
    </div>
    <div
      id="rightDiv"
      class="w-1/3 bg-blue-200 flex justify-center">
      <ul>
        <li
          v-for="(div, index) in divs"
          :key="'list-' + index">
          {{ div.name }}
          <button
            v-if="div.type !== 'video'"
            @click="removeDiv(index)">
            刪除
          </button>

          <div v-if="div.type === 'video'">
            <button @click="playVideo(index)">播放</button>
            <button @click="pauseVideo(index)">暫停</button>
            <button @click="stopVideo(index)">停止</button>
            <button @click="restartVideo(index)">重新開始</button>
            <button @click="removeDiv(index)">刪除</button>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref } from 'vue';

const divs: Ref<Div[]> = ref([]);
const isDragging = ref(false);
const startX = ref(0);
const startY = ref(0);

const currentDraggingIndex = ref(-1);

const containerRect: Ref<Rect | null> = ref(null);

const showAddVideoModal = ref(false);
const videoUrl = ref('');

interface Div {
  name: string;
  x: number;
  y: number;
  url?: string;
  type?: string;
}

interface Rect {
  left: number;
  top: number;
  right: number;
  bottom: number;
}

onMounted(() => {
  const container = document.querySelector('#leftDiv') as HTMLElement;
  if (container) {
    const rect = container.getBoundingClientRect();
    containerRect.value = { left: rect.left, top: rect.top, right: rect.right, bottom: rect.bottom };
  }
});

function addDiv() {
  const newDiv: Div = {
    name: `Div ${divs.value.length + 1}`,
    x: 50,
    y: 50,
  };
  divs.value.push(newDiv);
}

function openAddVideoModal() {
  showAddVideoModal.value = true;
}

function addVideo() {
  if (!videoUrl.value.includes('.mp4')) {
    alert('請輸入正確的影片網址');
    return;
  }
  divs.value.push({
    name: `Video ${divs.value.length + 1}`,
    x: 100,
    y: 100,
    url: videoUrl.value,
    type: 'video',
  });
  showAddVideoModal.value = false;
  videoUrl.value = '';
}

function playVideo(index: number) {
  const videoElement = document.getElementById(`video-${index}`) as HTMLVideoElement;
  videoElement.play();
}

function pauseVideo(index: number) {
  const videoElement = document.getElementById(`video-${index}`) as HTMLVideoElement;
  videoElement.pause();
}

function stopVideo(index: number) {
  const videoElement = document.getElementById(`video-${index}`) as HTMLVideoElement;
  videoElement.pause();
  videoElement.currentTime = 0;
}

function restartVideo(index: number) {
  const videoElement = document.getElementById(`video-${index}`) as HTMLVideoElement;
  videoElement.currentTime = 0;
  videoElement.play();
}

function removeDiv(index: number) {
  divs.value.splice(index, 1);
}

function startDrag(event: MouseEvent | TouchEvent, index: number): void {
  event.preventDefault();

  document.body.classList.add('no-select');

  currentDraggingIndex.value = index;
  isDragging.value = true;

  const clientX = event.type.includes('mouse')
    ? (event as MouseEvent).clientX
    : (event as TouchEvent).touches[0].clientX;
  const clientY = event.type.includes('mouse')
    ? (event as MouseEvent).clientY
    : (event as TouchEvent).touches[0].clientY;

  startX.value = clientX;
  startY.value = clientY;

  document.addEventListener('touchmove', doDrag as EventListener, { passive: false });
  document.addEventListener('touchend', stopDrag as EventListener);
  document.addEventListener('mousemove', doDrag as EventListener);
  document.addEventListener('mouseup', stopDrag as EventListener);
}

function doDrag(event: MouseEvent | TouchEvent): void {
  event.preventDefault();
  document.body.classList.add('no-select');
  if (!isDragging.value || currentDraggingIndex.value === -1 || !containerRect.value) return;

  const clientX = event instanceof MouseEvent ? event.clientX : event.touches[0].clientX;
  const clientY = event instanceof MouseEvent ? event.clientY : event.touches[0].clientY;

  const dx = clientX - startX.value;
  const dy = clientY - startY.value;

  let newX = divs.value[currentDraggingIndex.value].x + dx;
  let newY = divs.value[currentDraggingIndex.value].y + dy;

  const containerWidth = containerRect.value.right - containerRect.value.left;
  const buttonHeight = 48;

  const containerHeight = 800;

  const divWidth = 320;
  const divHeight = 320 - buttonHeight;

  newX = Math.max(0, Math.min(newX, containerWidth - divWidth));
  newY = Math.max(buttonHeight, Math.min(newY, containerHeight - divHeight));

  divs.value[currentDraggingIndex.value].x = newX;
  divs.value[currentDraggingIndex.value].y = newY;

  startX.value = event instanceof MouseEvent ? event.pageX : event.touches[0].clientX;
  startY.value = event instanceof MouseEvent ? event.pageY : event.touches[0].pageY;
}

function stopDrag() {
  document.body.classList.remove('no-select');
  isDragging.value = false;

  document.removeEventListener('touchmove', doDrag as EventListener);
  document.removeEventListener('touchend', stopDrag as EventListener);
  document.removeEventListener('mousemove', doDrag as EventListener);
  document.removeEventListener('mouseup', stopDrag as EventListener);

  currentDraggingIndex.value = -1;
}

function showAlert() {
  alert('你點擊div 兩次');
}
</script>

<style scoped>
.no-select {
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

.flex {
  display: flex;
}

#leftDiv,
#rightDiv {
  padding: 20px;
}

button {
  padding: 10px 15px;
  background-color: #4caf50;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  transition: background-color 0.3s ease;
}

button:hover {
  background-color: #45a049;
}

#videoControls button {
  padding: 5px 10px;
  margin-right: 10px;
}

input {
  margin: 5px 0;
  padding: 10px;
  border: 1px solid #ccc;
  border-radius: 5px;
}

.bg-red-200,
video {
  border: 2px solid #ccc;
  border-radius: 5px;
  margin-bottom: 10px;
}

.showAddVideoModal {
  position: fixed;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
  background-color: white;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 5px 15px rgba(0, 0, 0, 0.3);
  z-index: 1000;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  margin-bottom: 10px;
}
</style>
