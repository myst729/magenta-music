<script setup lang="ts">
import { onMounted, ref } from 'vue'
import { TWINKLE_TWINKLE, DRUMS } from './assets/note-sequences'
import { sequences, Player, MusicRNN, MusicVAE } from '@magenta/music/es6'

let music_rnn: any
let music_vae: any
let rAF = { handler: -1, timestamp: -1 }
const player = new Player()
const selected = ref<string>()
const state = ref<string>('')
const stateIndex = ref<number>(-1)

const loop = () => {
  if (player.isPlaying()) {
    if (Date.now() - rAF.timestamp > 200) {
      stateIndex.value++
      state.value = ' ' + `\\|/-`[stateIndex.value % 4]
      rAF.timestamp = Date.now()
    }
    rAF.handler = requestAnimationFrame(loop)
  } else {
    state.value = ''
    cancelAnimationFrame(rAF.handler)
  }
}

const handleChange = (value: string) => {
  selected.value = value
}

const handleClick = async () => {
  if (player.isPlaying()) {
    player.stop()
  } else {
    switch (selected.value) {
      case 'twinkle':
        player.start(TWINKLE_TWINKLE)
        break
      case 'drums':
        player.start(DRUMS)
        break
      case 'rnn':
        const qns = sequences.quantizeNoteSequence(TWINKLE_TWINKLE, 4)
        const sampleRnn = await music_rnn.continueSequence(qns, 20, 1.5)
        player.start(sampleRnn)
        break
      case 'vae':
        const [sampleVae] = await music_vae.sample(1, 1.5)
        player.start(sampleVae)
        break
      default:
        break
    }
    rAF.timestamp = Date.now()
    loop()
  }
}

onMounted(() => {
  music_rnn = new MusicRNN('https://storage.googleapis.com/magentadata/js/checkpoints/music_rnn/basic_rnn')
  music_vae = new MusicVAE('https://storage.googleapis.com/magentadata/js/checkpoints/music_vae/mel_4bar_small_q2')
  music_rnn.initialize()
  music_vae.initialize()
})
</script>

<template>
  <div>
    <a href="https://vite.dev" target="_blank">
      <img src="/logo.svg" class="logo" />
    </a>
  </div>
  <div class="options">
    <label><input type="radio" name="sequences" @change="handleChange('twinkle')" /> Twinkle Twinkle</label>
    <label><input type="radio" name="sequences" @change="handleChange('drums')" /> Drums</label>
    <label><input type="radio" name="sequences" @change="handleChange('rnn')" /> RNN</label>
    <label><input type="radio" name="sequences" @change="handleChange('vae')" /> VAE</label>
  </div>
  <div class="card">
    <button @click="handleClick" :disabled="!selected">{{ player.isPlaying() ? 'Stop' : 'Play' }}{{ state }}</button>
  </div>
</template>

<style scoped>
.logo {
  height: 8em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
.options {
  display: flex;
  gap: 1rem;
}
label {
  cursor: pointer;
}
</style>
