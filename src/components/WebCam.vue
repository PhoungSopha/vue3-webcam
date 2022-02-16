<script setup>
import { ref } from "vue"

const isOpenCamera = ref(false)
const isLoading = ref(false)
const isShotPhoto = ref(false)
const isPhotoTaken = ref(false)
const camera = ref(null)
const canvas = ref(null)

function toggleCamera() {
  if (isOpenCamera.value) {
    isOpenCamera.value = false
    isPhotoTaken.value = false
    isShotPhoto.value = false
    stopCameraStream()
  } else {
    isOpenCamera.value = true
    createCameraElement()
  }
}
function createCameraElement() {
  isLoading.value = true

  const constraint = (window.constraints = { audio: false, video: true })
  navigator.mediaDevices
    .getUserMedia(constraint)
    .then((stream) => {
      isLoading.value = false
      camera.value.srcObject = stream
      console.log(stream)
    })
    .catch((error) => {
      isLoading.value = false
      alert("May the browser did not support")
    })
}
function stopCameraStream() {
  let tracks = camera.value.srcObject.getTracks()
  tracks.forEach((track) => {
    track.stop()
  })
}
function takePhoto() {
  if (!isPhotoTaken.value) {
    isShotPhoto.value = true

    setTimeout(() => {
      isShotPhoto.value = false
    }, 50)
  }

  isPhotoTaken.value = !isPhotoTaken.value

  const context = canvas.value.getContext("2d")
  context.drawImage(camera.value, 0, 0, 450, 337.5)
}
function downloadPhoto() {
  const link = document.createElement("a")
  const canvas = document.getElementById("photoTaken").toDataURL("image/png").replace("image/png", "image/octet-stream")
  link.setAttribute("href", canvas)
  link.setAttribute("download", "Picture.png")
  link.style.display = "none"
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}
</script>
<template>
  <div class="min-h-screen min-w-full bg-emerald-500 flex items-center justify-center">
    <div class="bg-white w-full sm:h-[50rem] sm:w-[50rem] rounded-md shadow-sm">
      <div class="flex flex-col space-y-2 items-center p-4">
        <div class="p-3">
          <button class="rounded-full px-4 py-2 text-white font-semibold" :class="{ 'bg-emerald-500': !isOpenCamera, 'bg-red-500': isOpenCamera }" @click="toggleCamera"><span v-if="!isOpenCamera">Open Camera</span> <span v-else>Close Camera</span></button>
        </div>

        <div v-show="isOpenCamera && isLoading">
          <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 animate-spin-slow transform rotate-180 stroke-slate-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 4v5h.582m15.356 2A8.001 8.001 0 004.582 9m0 0H9m11 11v-5h-.581m0 0a8.003 8.003 0 01-15.357-2m15.357 2H15" />
          </svg>
        </div>

        <div v-if="isOpenCamera" v-show="!isLoading" :class="{ 'opacity-100': isShotPhoto }" class="w-full">
          <!-- <div class="w-24 h-24 opacity-0 bg-white absolute my-2 border-1" :class="{ 'opacity-100': isShotPhoto }"></div> -->
          <video v-show="!isPhotoTaken" ref="camera" class="w-[40rem] my-2 mx-auto rounded" autoplay></video>
          <canvas v-show="isPhotoTaken" id="photoTaken" :width="450" :height="337.5" ref="canvas" class="w-[40rem] h-full mx-auto rounded"></canvas>
        </div>

        <div v-if="isOpenCamera && !isLoading">
          <button class="h-14 w-14 bg-white rounded-full border-2 flex justify-center items-center hover:bg-gray-50" @click="takePhoto">
            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M4 5a2 2 0 00-2 2v8a2 2 0 002 2h12a2 2 0 002-2V7a2 2 0 00-2-2h-1.586a1 1 0 01-.707-.293l-1.121-1.121A2 2 0 0011.172 3H8.828a2 2 0 00-1.414.586L6.293 4.707A1 1 0 015.586 5H4zm6 9a3 3 0 100-6 3 3 0 000 6z" clip-rule="evenodd" />
            </svg>
          </button>
        </div>

        <div v-if="isPhotoTaken && isOpenCamera" class="pt-5">
          <button class="px-4 py-2 rounded-full bg-emerald-500 text-white font-semibold hover:bg-emerald-600" @click="downloadPhoto">Download</button>
        </div>
      </div>
    </div>
  </div>
</template>
