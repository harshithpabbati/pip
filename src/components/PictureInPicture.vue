<template>
  <div class="picture-in-picture">
    <div id="pip" class="pip">
      <video ref="videoRef" id="pip-video" autoplay playsinline muted></video>
    </div>
    <canvas id="picture-in-picture" width="1280" height="720"></canvas>
  </div>
  <div class="pip-wrapper">
    <button class="pip-button" @click="togglePIP">
      PIP
    </button>
  </div>
</template>

<script>
export default {
  name: "PictureInPicture",
  props: ['participants'],
  data() {
    return {
      canvas: null,
      videos: [],
    };
  },
  mounted() {
    this.videoRef = this.$refs.videoRef;
    this.canvas = document.getElementById('picture-in-picture');
    this.ctx = this.canvas.getContext('2d');
    this.handlePIP();
  },
  updated() {
    this.handlePIP();
  },
  methods: {
    handleVideo(participant) {
      if (!participant?.video) return;
      const videoTrack = participant?.tracks?.video?.persistentTrack;
      const source = new MediaStream([videoTrack]);
      return source;
    },
    handlePIP () {
      this.videos = Array.from(document.querySelectorAll('.tile-container video'));
      requestAnimationFrame(this.drawFrame);
      this.videoRef.srcObject = this.canvas.captureStream();
      this.videoRef.play();
    },
    drawFrame() {
      this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);

      const aspectRatio = 16 / 9;
      let maxColumns, maxRows, videoHeight, videoWidth;
      if (this.videos.length > 4) {
        maxColumns = 3;
        maxRows = Math.ceil(this.videos.length / maxColumns);
        videoWidth = this.canvas.width / maxColumns;
        videoHeight = this.canvas.height / maxRows;
      } else {
        maxColumns = this.videos.length === 1 ? 1: 2;
        maxRows = Math.ceil(this.videos.length / maxColumns);
        videoWidth = this.canvas.width / maxColumns;
        videoHeight = videoWidth / aspectRatio;
      }

      for (let i = 0; i < this.videos.length; i++) {
        const video = this.videos[i];
        const x = (i % maxColumns) * videoWidth;
        const y = maxRows === 1 ? (this.canvas.height - videoHeight) / 2 :Math.floor(i / maxColumns) * videoHeight;
        this.ctx.drawImage(video, x, y, videoWidth, videoHeight);
      }

      requestAnimationFrame(this.drawFrame);
    },
    togglePIP () {
      if(document.pictureInPictureEnabled && !this.videoRef.disablePictureInPicture) {
        try {
          if (document.pictureInPictureElement) {
            document.exitPictureInPicture();
          }
          this.videoRef.requestPictureInPicture();
        } catch(err) {
            console.error(err);
        }
      }
    }
  },
};
</script>

<style scoped>
.picture-in-picture {
  opacity: 0;
}
.pip-wrapper {
  position: absolute;
  top: 40%;
  right: 0;
  z-index: 99;
}
.pip-button {
  background-color: #fff;
  border: none;
  cursor: pointer;
  border-radius: 16px 0 0 16px;
  padding: 16px 14px 13px 18px;
}
</style>