<template>
  <div id="app">
    <input type="file" v-on:change="loadSong">
    <audio id="audio" controls></audio>
    <button v-on:click="play">Play</button>
    <button v-on:click="pause">Pause</button>
    <button v-on:click="circleTest">Circle</button>
    <button v-on:click="lineTest">Line</button>
    <button v-on:click="lyricsTest">Lyrics</button>
    <div id="lyrics">
      <span v-for="beat in lyrics" class="lyric">{{beat}}</span>
    </div>
  </div>
</template>

<script>
import songFuncs from './songFuncs.json'
import _ from 'lodash'

let lyrics = `I wake up fine and dan+dy but then by the time I find it han+dy To rip my heart a+part and start Plan+ning my crash lan+ding I go up, up, up, up, up to the cei+ling Then I feel my soul start lea+ving Like an old man's hair re+ce+ding I'm plea+ding, please, oh please On my knees re+pea+ted+ly as+king Why it's got to be like this Is this li+ving free? I don't wan+na be the one be the one to have the sun's blood on my hands I'll tell the moon Take this wea+pon, forged in dark+ness Some see a pen, I see a har+poon`.toUpperCase()

let splitLyrics = _.flatten(lyrics.split(/[ ]+/).map(word => word.split('+')).map(wordArr => wordArr.concat(' ')))

let data = {
  audio: document.getElementById('audio'),
  timeouts: [],
  lyrics: splitLyrics
}

let lyricCounter = 0

let visuals = {
  circles (key) {
    let el = document.createElement('div')
    el.classList = ['circle']
    let width = window.innerWidth
    let height = window.innerHeight
    document.querySelector('body').append(el)
    el.style.top = Math.random() * height - 150 + 'px'
    el.style.left = Math.random() * width - 150 + 'px'
  },
  lines (key) {
    let keyHeights = ['a', 's', 'd', 'f', 'j', 'k', 'l', ';']
    let height = window.innerHeight
    let el = document.createElement('div')
    el.classList = ['line']
    document.querySelector('body').append(el)
    el.style.top = (keyHeights.length - keyHeights.indexOf(key) - 1) * (height / keyHeights.length) + 'px'
  },
  lyrics (key) {
    let lyrics = document.getElementById('lyrics').children
    while (lyrics[lyricCounter].innerHTML == ' ') {
      lyricCounter++
    }
    let lyric = lyrics[lyricCounter]
    lyric.classList = ['lyric visible']
    lyricCounter++

    let lyricsWrapper = document.getElementById('lyrics')
    let trans = -lyric.getClientRects()[0].bottom + lyricsWrapper.getClientRects()[0].top
    lyricsWrapper.style.transform = `translateY(${trans}px)`
  }
}

function createVisualTimeout(func, call, visuals, audio, timeoutList) {
  timeoutList.push(setTimeout(() => {
    visuals[func.name](call.key)
  }, (call.playbackPosition - audio.currentTime / audio.duration) * audio.duration * 1000))
}

export default {
  name: 'app',
  data () {
    return data
  },
  methods: {
    loadSong (e) {
      let file = e.target.files[0]
      let audioReader = new FileReader()
      audioReader.onloadend = (event) => {
        if (event.target.readyState != FileReader.DONE) {
          throw Error("The song didn't load right :(")
        }
        this.audio = document.getElementById('audio')
        this.audio.setAttribute('src', event.target.result)
      }
      audioReader.readAsDataURL(file)
    },
    play () {
      this.audio.play()
      songFuncs.forEach(func => {
        func.calls.forEach(call => {
          createVisualTimeout(func, call, visuals, this.audio, this.timeouts)
        })
      })
    },
    pause () {
      this.audio.pause()
      while (this.timeouts.length > 0) {
        clearTimeout(this.timeouts.pop())
      }
    },
    circleTest () {
      visuals.circles('a')
    },
    lineTest () {
      visuals.lines('a')
    },
    lyricsTest () {
      visuals.lyrics('a')
    }
  }
}
</script>

<style>
body {
  background-color: black;
  color: white;
}

.circle {
  z-index: -9999;
  position: fixed;
  width: 300px;
  height: 300px;
  background-color: white;
  border-radius: 9999px;
  animation: shrink-out .4s linear forwards;
}

@keyframes shrink-out {
  0% {
    transform: scale(1);
  }
  100% {
    transform: scale(0);
  }
}

.line {
  z-index: -9999;
  position: fixed;
  left: 0;
  width: 100vw;
  height: 10vh;
  background-color: red;
  animation: fade-out .4s linear forwards;
}

@keyframes fade-out {
  0% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

#lyrics {
  z-index: -9998;
  position: fixed;
  top: 100vh;
  left: 0;
  width: 100vw;
  font-family: 'arial';
  font-style: bold;
  font-size: 100px;
  color: gray;
  transition: transform .4s ease;
  transform: translateY(-100px);
}

.lyric {
  opacity: 0;
}

.lyric.visible {
  opacity: 1;
  transition: opacity .2s linear;
}

@keyframes fade-in {
  0% {
    opacity: 0;
  }
  100% {
    opacity: 1;
  }
}

</style>
