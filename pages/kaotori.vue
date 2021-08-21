
<template>
  <v-row>
    <v-col>
      <v-card width="640" height="480">
        <video
          ref="video"
          width="640"
          autoplay
          playsinline
          muted
          style="position: relative: position: absolute"
        />
        <canvas
          id="camera-canvas"
          width="640"
          height="480"
          style="position: relative; position: absolute"
        />
      </v-card>
    </v-col>
    <v-col>
      <v-card width="640">
        <canvas id="capture-canvas" width="160" height="240" />
        <v-card-actions>
          <v-btn @click="capture()"> Capture </v-btn>
        </v-card-actions>
        <v-divider />
        <v-card-text>
          {{ classScore ? classScore : 'No Face Detected.' }}
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import * as faceapi from 'face-api.js'

export default {
  data() {
    return {
      video: null,
      classScore: null,
      boxTopLeftx: null,
      boxTopLefty: null,
      boxWidth: null,
      boxHeight: null,
      intervals: [],
    }
  },
  mounted() {
    this.initializeCanvas()

    if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
      this.video = this.$refs.video
      this.initializeFaceLandmarkModel()

      navigator.mediaDevices.getUserMedia({ video: true }).then((stream) => {
        this.video.srcObject = stream
      })
      this.intervals.push(setInterval(this.updateCanvas, 1000))
    }
  },
  destroyed() {
    clearInterval(...this.intervals)
  },
  methods: {
    initializeCanvas() {
      const canvas = document.getElementById('camera-canvas')
      if (!canvas.getContext) return
      const ctx = canvas.getContext('2d')

      ctx.clearRect(0, 0, canvas.width, canvas.height)
    },

    async initializeFaceLandmarkModel() {
      await faceapi.nets.tinyFaceDetector.load('weights/')
      await faceapi.nets.faceLandmark68TinyNet.load('weights/')
    },

    async updateCanvas() {
      this.initializeCanvas()

      const canvas = document.getElementById('camera-canvas')
      if (!canvas.getContext) return
      const ctx = canvas.getContext('2d')

      const result = await faceapi
        .detectSingleFace(this.video, new faceapi.TinyFaceDetectorOptions())
        .withFaceLandmarks(true)
      if (!result) return

      const margin = result.detection.box.height * 0.25
      this.boxTopLeftx = result.detection.box.topLeft.x
      this.boxTopLefty = result.detection.box.topLeft.y - margin
      this.boxWidth = result.detection.box.width
      this.boxHeight = result.detection.box.height + margin
      this.classScore = result.detection.classScore

      ctx.strokeRect(
        this.boxTopLeftx,
        this.boxTopLefty,
        this.boxWidth,
        this.boxHeight
      )

      for (let i = 0; i < result.landmarks.positions.length; i++) {
        ctx.strokeRect(
          result.landmarks.positions[i].x,
          result.landmarks.positions[i].y,
          1,
          1
        )
      }
    },
    capture() {
      const canvas = document.getElementById('capture-canvas')
      if (!canvas.getContext) return
      const ctx = canvas.getContext('2d')

      ctx.drawImage(
        this.video,
        this.boxTopLeftx,
        this.boxTopLefty,
        this.boxWidth,
        this.boxHeight,
        0,
        0,
        canvas.width,
        canvas.height
      )
    },
  },
}
</script>
