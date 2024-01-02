<template>
  <div id="app">
    <header>
      <h1>🎵 Lofi Music 🎵</h1>
    </header>

    <main>
      <section class="player">
        <h2 class="song-title">
          {{ current.title }} - <span>{{ current.artist }}</span>
        </h2>
        <div :key="current.src" class="image-container">
          <img v-if="current.cover" :src="current.cover" alt="Music Cover" class="music-cover"
            :class="{ 'loaded': imageLoaded }" @load="imageLoaded = true" />
        </div>
        <div class="progress-bar" @click="updateProgressBar">
          <div class="progress" :style="{ width: (currentTime / duration) * 100 + '%' }"></div>
        </div>

        <div class="time-display">
          {{ formatTime(currentTime) }} / {{ formatTime(player.duration || 0) }}
        </div>
        <div class="controls">
          <button class="stop" @click="stop">
            <i class="fas fa-stop"></i>
          </button>
          <button class="prev" @click="prev">
            <i class="fas fa-backward"></i>
          </button>
          <button class="play" v-if="!isPlaying" @click="play">
            <i class="fas fa-play"></i>
          </button>
          <button class="pause" v-else @click="pause">
            <i class="fas fa-pause"></i>
          </button>
          <button class="next" @click="next">
            <i class="fas fa-forward"></i>
          </button>
          <button class="repeat" @click="toggleRepeat" :class="{ active: isRepeat }">
            <i class="fas fa-redo"></i>
          </button>
        </div>

        <section class="playlist">
          <h3>Playlist</h3>
          <button v-for="song in songs" :key="song.src" @click="play(song)"
            :class="song.src == current.src ? 'song playing' : 'song'">
            {{ song.title }} - {{ song.artist }}
          </button>
        </section>
      </section>
    </main>
  </div>
</template>

<script>
export default {
  name: "App",
  data() {
    return {
      current: {},
      index: 0,
      isPlaying: false,
      currentTime: 0,
      duration: 0,
      songs: [
        {
          title: "Morning Routine",
          artist: "Lofi Study Music",
          src: require("./assets/Morning-Routine-Lofi-Study-Music.mp3"),
          cover: require("./assets/images/lofi-image.png"),
        },
        {
          title: "Still-Awake",
          artist: "Lofi Music",
          src: require("./assets/Still-Awake-Lofi-Study-Music.mp3"),
          cover: require("./assets/images/lofi-image2.jpg"),

        },
        {
          title: "On My Way",
          artist: "Lofi Studio",
          src: require("./assets/On-My-Way-Lofi-Study-Music.mp3"),
          cover: require("./assets/images/lofi-image3.jpg"),

        },
      ],
      player: new Audio(),
      imageLoaded: false,
      isRepeat: false,
    };
  },
  methods: {
    play(song) {
      if (typeof song.src != "undefined") {
        this.transitionCover();
        this.current = song;
        this.player.src = this.current.src;
      }

      this.duration = this.player.duration;
      this.currentTime = this.player.currentTime;

      this.timer = setInterval(() => {
        this.currentTime = this.player.currentTime;

        // Ajuster la barre de progression
        if (this.player.currentTime >= this.duration) {
          clearInterval(this.timer);
        }
      }, 100);

      this.player.play();

      // musique terminé passe suivante
      this.player.addEventListener(
        "ended",
        () => {
          clearInterval(this.timer);
          if (this.isRepeat) {
            // Répéter la musique
            this.currentTime = 0;
            this.current = this.songs[this.index];
            this.play(this.current);


          } else {
            // Passer à la musique suivante
            this.index++;
            if (this.index > this.songs.length - 1) {
              this.index = 0;
            }
            this.current = this.songs[this.index];
            this.currentTime = 0;
            this.play(this.current);
          }

        }
      );

      this.isPlaying = true;
    },
    updateProgressBar(event) {
      //changer la position de la musique en fonction de la position de la souris
      const progressBar = this.$el.querySelector('.progress-bar');
      const newTime = (event.offsetX / progressBar.clientWidth) * this.duration;
      this.player.currentTime = newTime;
      this.currentTime = newTime;
      this.duration = this.player.duration;
    },

    pause() {
      this.player.pause();
      this.isPlaying = false;
      clearInterval(this.timer);
    },
    next() {
      this.index++;
      if (this.index > this.songs.length - 1) {
        this.index = 0;
      }
      this.current = this.songs[this.index];
      this.play(this.current);

      clearInterval(this.timer);
      this.duration = this.player.duration;
      this.transitionCover();

    },
    prev() {
      this.index--;
      if (this.index < 0) {
        this.index = this.songs.length - 1;
      }
      this.current = this.songs[this.index];
      this.play(this.current);
      clearInterval(this.timer);
      this.duration = this.player.duration;
      this.transitionCover();

    },

    toggleRepeat() {
      this.isRepeat = !this.isRepeat;
    },

    stop() {
      this.player.pause();
      this.player.currentTime = 0;
      this.isPlaying = false;
      clearInterval(this.timer);
      this.currentTime = 0;
      this.duration = this.player.duration;
    },
    formatTime(time) {
      const minutes = Math.floor(time / 60);
      const seconds = Math.floor(time % 60);
      return `${minutes}:${seconds < 10 ? '0' : ''}${seconds}`;
    },
    transitionCover() {
      if (!this.isRepeat)
        this.imageLoaded = false;
      setTimeout(() => {
        this.imageLoaded = true;
      }, 10);
    },
  },
  created() {
    this.current = this.songs[this.index];
    this.player.src = this.current.src;
  },
  mounted() {
    // Récupérer la durée de la musique une fois qu'elle est chargée
    this.player.addEventListener('loadedmetadata', () => {
      this.duration = this.player.duration;
    });
  },
};
</script>

<style>
*,
::before,
::after {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

:root {
  --couleur-principale: #1db954;
  --couleur-secondaire: #1ed760;
}

body {
  font-family: sans-serif;
}

header {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 55px;
  background: var(--couleur-principale);
  color: #fff;

}

header h1 {
  font-size: 40px;
  font-weight: 700;
  text-transform: uppercase;
}

main {
  width: 100%;
  max-width: 768px;
  margin: 0 auto;
  padding: 55px;
}

.song-title {
  color: #53565a;
  font-size: 32px;
  font-weight: 700;
  text-transform: uppercase;
  text-align: center;
}

.song-title span {
  font-weight: 500;
  font-style: italic;
}

.controls {
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 30px 15px;
}

button {
  appearance: none;
  background: none;
  border: none;
  outline: none;
  cursor: pointer;
}

button:hover {
  opacity: 0.8;
}

.play,
.pause {
  font-size: 20px;
  font-weight: 700;
  padding: 15px 25px;
  margin: 0px 15px;
  border-radius: 8px;
  color: #fff;
  background-color: var(--couleur-principale);
}

.next,
.prev {
  font-size: 16px;
  font-weight: 700;
  padding: 10px 20px;
  margin: 0px 15px;
  border-radius: 6px;
  color: #fff;
  background-color: var(--couleur-secondaire);
}

.playlist {
  padding: 0px 30px;
  background: #212121;
  padding: 5px 10px;
}

.playlist h3 {
  color: #fff;
  font-size: 40px;
  font-weight: bold;
  margin-bottom: 30px;
  margin-top: 30px;
  text-align: center;
}

.playlist .song {
  display: block;
  width: 100%;
  padding: 15px;
  font-size: 20px;
  font-weight: 700;
  cursor: pointer;
  color: #53565a;
}

.playlist .song:hover {
  color: var(--couleur-secondaire);
}

.playlist .song.playing {
  color: #fff;
  background-image: linear-gradient(to right,
      var(--couleur-principale),
      var(--couleur-secondaire));
}

.progress-bar {
  height: 10px;
  background-color: #ccc;
  position: relative;
  cursor: pointer;
}

.progress {
  height: 100%;
  width: 0;
  background-color: var(--couleur-secondaire);
  transition: width 0.3s ease;
}

.time-display {
  text-align: center;
  font-size: 14px;
  font-weight: 700;
  color: #53565a;
  margin-top: 10px;
}

.image-container {
  height: 400px;
  overflow: hidden;
  margin-top: 20px;
  margin-bottom: 20px;
  text-align: center;
  transition: opacity 0.5s ease;
}

.music-cover {
  width: 100%;
  height: 100%;
  object-fit: cover;
  opacity: 0;
  transition: opacity 0.5s ease;
}

.music-cover.loaded {
  opacity: 1;
}


.repeat {
  font-size: 16px;
  font-weight: 700;
  padding: 10px 20px;
  margin: 0px 15px;
  border-radius: 6px;
  background-color: #53565a;
  color: #fff;
}

.repeat.active i {
  color: var(--couleur-principale);
}

.stop {
  font-size: 16px;
  font-weight: 700;
  padding: 10px 20px;
  margin: 0px 15px;
  border-radius: 6px;
  color: #fff;
  background-color: #53565a;
}

.stop:hover {
  background-color: #53565a;
}

/* media queries mobile */
@media (max-width: 768px) {
  header {
    padding: 30px;
  }

  header h1 {
    font-size: 30px;
  }

  main {
    padding: 30px;
  }

  .song-title {
    font-size: 24px;
  }

  .controls {
    padding: 15px;
  }

  .controls button {
    font-size: 10px;
  }

  .playlist h3 {
    font-size: 30px;
  }

  .playlist .song {
    font-size: 16px;
  }

  .time-display {
    font-size: 12px;
  }

  .image-container {
    height: 300px;
  }

  .music-cover {
    height: 100%;
  }

  .play,
  .pause {
    font-size: 16px;
    font-weight: 700;
    padding: 10px 20px;
    margin: 0px 15px;
    border-radius: 6px;
    color: #fff;
    background-color: var(--couleur-principale);
  }

  .next,
  .prev {
    font-size: 12px;
    font-weight: 700;
    padding: 7px 12px;
    margin: 0px 15px;
    border-radius: 6px;
    color: #fff;
    background-color: var(--couleur-secondaire);
  }

  .repeat,
  .stop {
    font-size: 12px;
    font-weight: 700;
    padding: 7px 12px;
    margin: 0px 15px;
    border-radius: 6px;
    color: #fff;
    background-color: #53565a;
  }


}
</style>
