<template>
    <div class="player-container">
      <input type="file" @change="loadMusic" multiple accept="audio/*" />
      <ul>
        <li v-for="(track, index) in tracks" :key="index" @click="playTrack(index)">
          {{ track.name }}
        </li>
      </ul>
      <audio ref="audioPlayer" @ended="nextTrack" @canplay="onAudioLoaded" @timeupdate="updateSeek"></audio>
      <canvas ref="canvas" width="400" height="150"></canvas>
      <div class="controls">
        <button @click="prevTrack">Prev</button>
        <button @click="togglePlay">{{ isPlaying ? 'Pause' : 'Play' }}</button>
        <button @click="nextTrack">Next</button>
      </div>
      <!-- Seek bar -->
      <input type="range" v-model="currentTime" :max="duration" @input="seek" />
      <span>{{ formatTime(currentTime) }} / {{ formatTime(duration) }}</span>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        tracks: [],
        currentTrackIndex: 0,
        isPlaying: false,
        audioContext: null,
        analyser: null,
        bufferLength: 0,
        dataArray: null,
        canvasContext: null,
        audioElement: null,
        currentTime: 0, // Current playback time
        duration: 0, // Duration of the audio track
      };
    },
    mounted() {
      this.audioElement = this.$refs.audioPlayer;
    },
    methods: {
      loadMusic(event) {
        const files = Array.from(event.target.files);
        this.tracks = files.map((file) => ({
          name: file.name,
          url: URL.createObjectURL(file),
        }));
        if (this.tracks.length > 0) {
          this.currentTrackIndex = 0;
        }
      },

      getRandomColor() {
        return Math.floor(Math.random() * 256);
        },
      playTrack(index) {
        this.currentTrackIndex = index;
        this.play();
      },
      play() {
        if (!this.audioElement) {
          console.error('Audio element is not available');
          return;
        }
  
        this.audioElement.src = this.tracks[this.currentTrackIndex].url;
        this.audioElement.play();
        this.isPlaying = true;
  
        this.setupAudioAnalyser();
        this.updateSpectrum();
      },
      togglePlay() {
        if (!this.audioElement) {
          console.error('Audio element is not available');
          return;
        }
  
        if (this.isPlaying) {
          this.audioElement.pause();
        } else {
          this.audioElement.play();
        }
        this.isPlaying = !this.isPlaying;
      },
      nextTrack() {
        if (!this.audioElement) {
          console.error('Audio element is not available');
          return;
        }
  
        if (this.currentTrackIndex < this.tracks.length - 1) {
          this.currentTrackIndex++;
        } else {
          this.currentTrackIndex = 0;
        }
        this.play();
      },
      prevTrack() {
        if (!this.audioElement) {
          console.error('Audio element is not available');
          return;
        }
  
        if (this.currentTrackIndex > 0) {
          this.currentTrackIndex--;
        } else {
          this.currentTrackIndex = this.tracks.length - 1;
        }
        this.play();
      },
      setupAudioAnalyser() {
        if (!this.audioElement) {
          console.error('Audio element is missing');
          return;
        }
  
        this.audioContext = new (window.AudioContext || window.webkitAudioContext)();
        this.analyser = this.audioContext.createAnalyser();
  
        const source = this.audioContext.createMediaElementSource(this.audioElement);
        source.connect(this.analyser);
        this.analyser.connect(this.audioContext.destination);
  
        this.analyser.fftSize = 256;
        this.bufferLength = this.analyser.frequencyBinCount;
        this.dataArray = new Uint8Array(this.bufferLength);
  
        this.canvasContext = this.$refs.canvas.getContext('2d');
      },
      updateSpectrum() {
        if (!this.audioElement || !this.canvasContext) {
          return;
        }
  
        this.analyser.getByteFrequencyData(this.dataArray);
  
        this.canvasContext.clearRect(0, 0, this.$refs.canvas.width, this.$refs.canvas.height);
  
        const barWidth = (this.$refs.canvas.width / this.bufferLength) * 2.5;
        let barHeight;
        let x = 0;
  
        for (let i = 0; i < this.bufferLength; i++) {
          barHeight = this.dataArray[i];
          this.canvasContext.fillStyle = `rgb(${barHeight + 100}, 50, 50)`;
          const randomColor = `rgb(${this.getRandomColor()}, ${this.getRandomColor()}, ${this.getRandomColor()})`;
        this.canvasContext.fillStyle = randomColor;
          this.canvasContext.fillRect(x, this.$refs.canvas.height - barHeight / 2, barWidth, barHeight);
          x += barWidth + 1;
        }
  
        if (this.isPlaying) {
          requestAnimationFrame(this.updateSpectrum);
        }
      },
      updateSeek() {
        if (this.audioElement) {
          this.currentTime = this.audioElement.currentTime;
          this.duration = this.audioElement.duration;
        }
      },
      seek() {
        if (this.audioElement) {
          this.audioElement.currentTime = this.currentTime;
        }
      },
      formatTime(seconds) {
        const minutes = Math.floor(seconds / 60);
        const remainingSeconds = Math.floor(seconds % 60);
        return `${minutes}:${remainingSeconds < 10 ? '0' : ''}${remainingSeconds}`;
      },
      onAudioLoaded() {
        this.setupAudioAnalyser();
      }
    },
  };
  </script>
  
  <style scoped>
  .player-container {
    text-align: center;
    max-width: 400px;
    margin: auto;
  }
  ul {
    list-style: none;
    padding: 0;
  }
  li {
    cursor: pointer;
    padding: 5px;
    border-bottom: 1px solid #ddd;
  }
  .controls {
    margin-top: 10px;
  }
  button {
    margin: 5px;
    padding: 10px;
  }
  canvas {
    margin-top: 10px;
    border: 1px solid #ddd;
  }
  input[type="range"] {
    width: 100%;
    margin-top: 10px;
  }
  </style>
  