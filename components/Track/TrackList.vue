<template>
  <ol class="list">
    <TrackListItem
      v-for="(track, i) in tracks.slice(0, itemsPerPage)"
      :key="i"
      :class="[
        'list__list-item',
        { 'list__list-item--border-bottom': showListOrder && i === tracks.length - 1 }
      ]"
      :show-order="showListOrder"
      :order="getOrder(i)"
      :item="track"
      :is-playing="getIsPlaying(track)"
      @click.native="$emit('playTrack', track.slug)"
    />
  </ol>
</template>

<script>
import _ from 'lodash'

import TrackListItem from './TrackListItem.vue'

export default {
  components: {
    TrackListItem
  },
  props: {
    showListOrder: {
      type: Boolean,
      defalut: false
    },
    currentSound: {
      type: Object,
      default() {
        return {}
      },
      required: true
    },
    isPlaying: {
      type: Boolean,
      default: false
    },
    tracks: {
      type: Array,
      required: true
    },
    isLatestFirst: {
      type: Boolean,
      default: true
    },
    page: {
      type: Number,
      default: 1
    },
    itemsPerPage: {
      type: Number,
      required: true
    },
    total: {
      type: Number,
      required: true
    }
  },
  methods: {
    getOrder(i) {
      if (this.isLatestFirst) {
        return this.total - i - this.itemsPerPage * (this.page - 1)
      } else {
        return i + 1 + this.itemsPerPage * (this.page - 1)
      }
    },
    getIsPlaying(track) {
      const slug = _.get(track, 'slug', '')
      return (
        this.isPlaying &&
        slug !== '' &&
        slug === _.get(this.currentSound, 'slug', '')
      )
    }
  }
}
</script>

<style lang="stylus" scoped>
.list
  list-style none
  margin 0
  padding 0
  &__list-item
    border-top 1px solid #eeeeee
    &--border-bottom
      border-bottom 1px solid #eeeeee
</style>
