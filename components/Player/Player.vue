<template>
  <div class="player">
    <audio ref="audio" />
    <div class="player__cover cover">
      <nuxt-link to="/single/fakeslug">
        <!-- <img class="cover__img" :src="currentSound.cover" alt> -->
      </nuxt-link>
    </div>
    <PlayerNavs
      class="player__navs"
      :show-nav="list.length > 1"
      :is-playing="isPlaying"
      @backward="playIndex -= 1"
      @pause="pause"
      @play="play"
      @forward="playIndex += 1"
    />
    <div class="player__middle middle">
      <PlayerInfo
        class="middle__info"
        :sound="currentSound"
        :duration="playStatDuration"
        :played="playStatPlayedTime"
      />
      <PlayerSlider
        class="middle__progress"
        :direction="'horizontal'"
        :buffered="(playStatLoadedTime / playStatDuration) || 0"
        :value="(playStatPlayedTime / playStatDuration) || 0"
        @valueChange="seek"
      />
    </div>
    <PlayerRate
      class="player__rate"
      :rate="currentPlaybackRate"
      @rateChange="v => currentPlaybackRate = v"
    />
    <PlayerVolume
      class="player__volume"
      :volume="currentVolume"
      @volumeChange="v => currentVolume = v"
    />
  </div>
</template>

<script>
import _ from 'lodash'

import PlayerNavs from './PlayerNavs.vue'
import PlayerInfo from './PlayerInfo.vue'
import PlayerRate from './PlayerRate.vue'
import PlayerVolume from './PlayerVolume.vue'
import PlayerSlider from './PlayerSlider.vue'

import initEventEmitters from './util/eventEmitters'
import initEventHandlers from './util/eventHandlers'

import { ratesAvailable } from './comm/rate'

export default {
  components: {
    PlayerNavs,
    PlayerInfo,
    PlayerRate,
    PlayerVolume,
    PlayerSlider
  },
  props: {
    sound: {
      type: Object,
      required: true,
      default() {
        return {
          title: '',
          src: '',
          slug: ''
        }
      }
    },
    list: {
      type: Array,
      default() {
        return []
      }
    },

    // native audio's properties as props
    volume: {
      type: Number,
      default: 1,
      validator(value) {
        return value >= 0 && value <= 1
      }
    },
    muted: {
      type: Boolean,
      default: false
    },
    playbackRate: {
      type: Number,
      default: 1.0,
      validator(value) {
        return ratesAvailable.includes(value)
      }
    },
    shouldPlaying: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      internalSound: this.sound,
      internalVolume: this.volume,
      internalMuted: this.muted,
      internalPlaybackRate: this.playbackRate,
      internalBufferedAmount: 0,
      internalPlayedAmount: 0,
      isPlaying: false,
      isLoading: false,
      playStatDuration: 0,
      playStatLoadedTime: 0,
      playStatPlayedTime: 0
    }
  },
  computed: {
    // alias of audio element
    audio() {
      return this.$refs.audio
    },

    playIndex: {
      get() {
        return _.findIndex(this.list, o => o.src === this.currentSound.src)
      },
      set(val) {
        if (val >= 0 && val <= this.list.length - 1) {
          this.currentSound = this.list[val]
        }
      }
    },

    // observers
    // inform parent component in setters
    currentSound: {
      get() {
        return this.internalSound
      },
      set(val) {
        this.$emit('update:sound', val)
        this.internalSound = val
      }
    },
    currentVolume: {
      get() {
        return this.internalVolume
      },
      set(val) {
        this.$emit('update:volume', val)
        this.internalVolume = val
      }
    },
    currentMuted: {
      get() {
        return this.internalMuted
      },
      set(val) {
        this.$emit('update:muted', val)
        this.internalMuted = val
      }
    },
    currentPlaybackRate: {
      get() {
        return this.internalPlaybackRate
      },
      set(val) {
        this.$emit('update:playbackRate', val)
        this.internalPlaybackRate = val
      }
    }
  },
  watch: {
    // observe mutable properties
    currentSound() {
      this.audio.src = this.currentSound.src
      this.play()
    },
    currentVolume() {
      this.audio.volume = this.currentVolume
    },
    currentMuted() {
      this.audio.muted = this.currentMuted
    },
    currentPlaybackRate() {
      this.audio.playbackRate = this.currentPlaybackRate
    },

    // sync muted, volume if parent component mutate the props
    sound() {
      this.internalSound = this.sound
    },
    volume() {
      this.internalVolume = this.volume
    },
    muted() {
      this.internalMuted = this.muted
    },
    playbackRate() {
      this.internalPlaybackRate = this.playbackRate
    },

    // coordinate volume and muted
    internalVolume() {
      this.internalMuted = this.internalVolume <= 0
    },
    internalMuted() {
      this.internalVolume = this.internalMuted ? 0 : this.volume
    },

    shouldPlaying() {
      this.shouldPlaying ? this.play() : this.pause()
    }
  },
  mounted() {
    this.initAudio()
  },
  methods: {
    initAudio() {
      // set exposed properties as their init value which is passing from props
      this.audio.volume = this.volume
      this.audio.muted = this.muted
      this.audio.playbackRate = this.playbackRate

      // init events
      initEventEmitters(this)
      initEventHandlers(this)

      // set audio's src
      if (this.currentSound) {
        this.audio.src = this.currentSound.src
      }
    },
    play() {
      this.audio.play()
    },
    pause() {
      this.audio.pause()
    },
    seek(percentage) {
      this.audio.currentTime = this.audio.duration * percentage
    }
  }
}
</script>

<style lang="stylus" scoped>
.player
  display flex
  align-items center
  height 60px
  max-width 1000px
  &__cover
    margin 0 31px 0 0
  &__middle
    margin 0 0 0 25px
  &__rate
    margin 0 0 0 20px
  &__volume
    margin 0 0 0 28px

.cover
  d = 34px
  width d
  height d
  &__img
    width 100%
    height 100%
    object-fit cover

.middle
  display flex
  flex-direction column
  width 700px
  &__progress
    margin 10px 0 0 0

@media (max-width 768px)
  .player
    max-width 100vw
    width 100vw
    &__cover
      display none
    &__middle
      order 1
      margin 0 0 0 10px
    &__rate
      order 0
    &__volume
      display none !important

  .middle
    display flex
    flex-direction column
    width 0px
    flex 1 1 auto
    &__progress
      margin 10px 0 0 0
</style>
