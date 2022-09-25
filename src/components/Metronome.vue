<template>
  <label for="volume">Volume</label>
  <input type="range" v-model="volume" id="volume" min="0" max="0.5" step="0.01" />

  <div class="flex space-x-8 justify-center items-center">
    <Button>-</Button>

    <input
      type="number"
      v-model="bpm"
      min="30"
      max="500"
      step="1"
      class="
        border-b-2
        border-lime-400
        px-5
        py-2.5
        text-2xl
        text-center
        w-24
      "
    />

    <Button>+</Button>
  </div>

  <div class="flex space-x-8 justify-center items-center">
    <Button>-</Button>

    <input
      type="number"
      v-model="meter"
      min="1"
      max="128"
      step="1"
      class="
        border-b-2
        border-lime-400
        px-5
        py-2.5
        text-2xl
        text-center
        w-24
      "
    />

    <Button>+</Button>
  </div>

  <div class="flex space-x-8 justify-center items-center">
    <Button>-</Button>

    <select
      v-model="subdivision"
      class="
        border-b-2
        border-lime-400
        px-5
        py-2.5
        text-2xl
        text-center
        w-24
      "
    >
      <option v-for="subdivision of subdivisions">
        {{ subdivision }}
      </option>
    </select>

    <Button>+</Button>
  </div>

  <div>
    <Button @click="toggleMetronome">
      {{ playButtonText }}
    </Button>
  </div>

  <div class="flex flex-wrap gap-8 justify-center items-center">
    <button
      v-for="i in meter"
      :key="i"
      class="
        w-16
        h-16
        rounded-lg
      "
      :class="currentBeat === i - 1 ? 'bg-red-400' : 'bg-lime-400'"
    />
  </div>
</template>

<script setup>
  import { computed, ref, watch } from 'vue'
  import Button from './common/Button.vue'

  const bpm = ref(100)
  const meter = ref(4)
  const subdivisions = ref([2, 4, 8, 16, 32, 64, 128])
  const subdivision = ref(4)
  const volume = ref(0.25)
  const playing = ref(false)
  const currentBeat = ref(null)

  let clickerId = null
  const audioContext = new AudioContext()
  const gainNode = audioContext.createGain()

  const playButtonText = computed(() => playing.value ? 'Pause' : 'Play')
  const subdivisionFactor = computed(() => subdivision.value / 4.0)

  gainNode.gain.value = volume.value

  watch(volume, newVolume => {
    gainNode.gain.value = newVolume
  })
  
  function toggleMetronome() {
    playing.value = !playing.value
    playing.value ? startMetronome({ time: audioContext.currentTime }) : stopMetronome()
  }

  function startMetronome({ time }) {
    let metronomeProps = {
      nextClickTime: audioContext.currentTime,
      currentBeat: 0,
    }

    clickerId = setInterval(() => {
      if (metronomeProps.nextClickTime < audioContext.currentTime + 0.1) {
        currentBeat.value = metronomeProps.currentBeat

        metronomeProps = scheduleClick({
          clickTime: metronomeProps.nextClickTime,
          currentBeat: metronomeProps.currentBeat,
        })
      }
    }, 25)
  }

  function stopMetronome() {
    clearInterval(clickerId)
    currentBeat.value = null
  }

  function scheduleClick({ clickTime, currentBeat }) {
    const click = audioContext.createOscillator()
    click.frequency.value = currentBeat === 0 ? 1000 : 500
    click.connect(gainNode).connect(audioContext.destination)

    click.start(clickTime)
    click.stop(clickTime + 0.1)

    return {
      nextClickTime: clickTime + (60.0 / bpm.value / subdivisionFactor.value),
      currentBeat: (currentBeat + 1) % meter.value,
    }
  }
</script>
