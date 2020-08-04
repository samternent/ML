<template>
  <div class="wrap">
    <section class="capture">
      <img src="/emoji.png" class="emoji" ref="emoji">
      <video :src="`/${videoSrc}.mp4`" ref="video" :style="{ opacity: opacity / 100 }"/>
      <canvas ref="canvasD"/>
      <canvas ref="canvasP"/>
    </section>
    <section class="controls">
      <button @click="startCapture">Start</button><br>
      <input type="range" min="0" max="100" v-model="opacity"/>
      <select v-model="videoSrc">
        <option
          v-for="video in videos"
          :key="video">{{ video }}</option>
      </select>
    </section>
  </div>
</template>

<script>
/* eslint-disable no-undef */
export default {
  name: 'HelloWorld',
  data: () => ({
    video: null,
    poses: [],
    detections: [],
    opacity: 10,
    videoSrc: 'ballet',
    videos: [
      'ballet',
      'football',
      'dogs',
      'pulpfiction',
    ],
  }),
  computed: {
  },
  async mounted() {
    this.$refs.video.setAttribute('width', window.innerWidth);
    this.$refs.video.setAttribute('height', window.innerHeight);
    this.$refs.canvasD.setAttribute('width', window.innerWidth);
    this.$refs.canvasD.setAttribute('height', window.innerHeight);
    this.$refs.canvasP.setAttribute('width', window.innerWidth);
    this.$refs.canvasP.setAttribute('height', window.innerHeight);

    this.$refs.video.setAttribute('muted', '');
    this.$refs.video.setAttribute('playsinline', '');

    // var constraints = {
    //      audio: false,
    //      video: {
    //          facingMode: 'user'
    //      }
    // }

    // const vm = this;
    // navigator.mediaDevices.getUserMedia(constraints).then(function success(stream) {
    //     vm.$refs.video.srcObject = stream;
    // });
      const vm = this;
      const poseNet = ml5.poseNet(this.$refs.video, () => {});
      // this.objectDetector = ml5.objectDetector('cocossd', {}, this.detect);
      poseNet.on('pose', (results) => {
        vm.poses = results;
      });

  },
  methods: {
    detect() {
      this.objectDetector.detect(this.$refs.video, this.gotResults);
    },
    gotResults(err, results) {
      if (err) {
        return
      }

      this.detections = results;

      this.detect();
    },
    startCapture() {
      // eslint-disable-next-line no-undef
      this.$refs.video.play();
    },
    stopCapture() {
      this.$refs.video.pause();
    },
  },
  watch: {
    detections() {
      const ctx = this.$refs.canvasD.getContext('2d');
      ctx.clearRect(0, 0, this.$refs.canvasD.width, this.$refs.canvasD.height);
      // ctx.lineWidth = 14;
      ctx.strokeStyle = '#2c3e50';
      const objects = this.detections;
      for (let i = 0; i < objects.length; i++) {

        ctx.font = "16px Arial";
        ctx.fillStyle = "green";
        ctx.fillText(objects[i].label, objects[i].x + 4, objects[i].y + 16);

        ctx.beginPath();
        ctx.rect(objects[i].x, objects[i].y, objects[i].width, objects[i].height);
        ctx.strokeStyle = "green";
        ctx.stroke();
        ctx.closePath();
      }
    },
    poses() {
      const ctx = this.$refs.canvasP.getContext('2d');
      ctx.clearRect(0, 0, this.$refs.canvasP.width, this.$refs.canvasP.height);
      // ctx.lineWidth = 14;
      ctx.strokeStyle = '#2c3e50';
      // Loop through all the skeletons detected
      // console.log(this.poses)
      for (let i = 0; i < this.poses.length; i++) {
        let pose = this.poses[i].pose;
        for (let j = 0; j < pose.keypoints.length; j++) {
          // A keypoint is an object describing a body part (like rightArm or leftShoulder)
          let keypoint = pose.keypoints[j];
          // Only draw an ellipse is the pose probability is bigger than 0.2
          if (keypoint.score > 0.2) {
            // if (keypoint.part === 'rightEye' || keypoint.part === 'leftEye') {
            //   // ctx.drawImage(this.$refs.emoji, 140, 70, 70, 70, keypoint.position.x - 20, keypoint.position.y - 20, 40, 40);
            // } else {
              ctx.beginPath();
              ctx.arc(keypoint.position.x, keypoint.position.y, 2, 0, 2 * Math.PI);
              ctx.stroke();
            // }
          }
        }
        // For every skeleton, loop through all body connections
        for (let j = 0; j < this.poses[i].skeleton.length; j++) {
          let partA = this.poses[i].skeleton[j][0];
          let partB = this.poses[i].skeleton[j][1];
          ctx.beginPath();
          ctx.moveTo(partA.position.x, partA.position.y);
          ctx.lineTo(partB.position.x, partB.position.y);
          ctx.stroke();
        }
      }
    },
  },
}
</script>
<style scoped>
.wrap {
  background-color: #333;
}
.capture {
  position: relative;
}
video,
canvas {
  position: fixed;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  z-index: 0;
}
video {
  z-index: 0;
  border: 2px solid white;
}
button {
  z-index: 10;
  background-color: rgba(0,0,0,0);
  padding: 7px 24px;
  border: 2px solid #2c3e50;
  font-size: 1.25em;
  color: #2c3e50;
  cursor: pointer;
  transition: background-color 0.1s;
}

.emoji {
  display: none;
}
button:hover {
  background-color: rgba(0,0,0,0.06);
}

.controls {
  position: fixed;
  text-align: center;
  padding: 30px;
  bottom: 0;
  left: 0; right: 0;
}
</style>
