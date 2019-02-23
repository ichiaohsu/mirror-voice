<template>
  <AppMainAsideWrapper>
    <div slot="main" class="main">
      <AppDiv class="main__wrapper info-wrapper">
        <Info
          :layout="'album'"
          :info="album"
        />
      </AppDiv>
      <AppDiv class="main__wrapper body-wrapper">
        <AppH1 class="body-wrapper__title">
          簡介
        </AppH1>
        <div class="body-wrapper__body" v-html="brief" />
      </AppDiv>
      <AppDiv class="main__wrapper tracks-wrapper">
        <DivHeader class="tracks-wrapper__header">
          <template slot="left">
            專輯音檔（{{ tracks.meta.total }}）
          </template>
          <div
            slot="right"
            class="sorts"
          >
            <button class="sorts__sort">
              最新
            </button>
            <button class="sorts__sort">
              最舊
            </button>
          </div>
        </DivHeader>
        <TrackList
          class="tracks-wrapper__tracks"
          :show-list-order="true"
          :tracks="tracks.items"
        />
        <AppPagination
          class="tracks-wrapper__pagination"
        />
      </AppDiv>
    </div>
    <div slot="aside" class="aside">
      <AsideIntro
        class="aside__wrapper anchor"
        :title="'主播'"
        :img-style="'round'"
      />
      <AsideAlbumList
        class="aside__wrapper anchor-relateds"
      />
    </div>
  </AppMainAsideWrapper>
</template>

<script>
import { mapState } from 'vuex'
import sanitizeHtml from 'sanitize-html'
import _ from 'lodash'

import AppMainAsideWrapper from '~/components/AppMainAsideWrapper.vue'
import AppDiv from '~/components/AppDiv.vue'
import AppH1 from '~/components/AppH1.vue'
import Info from '~/components/Info/Info.vue'
import DivHeader from '~/components/Div/DivHeader.vue'
import AsideIntro from '~/components/Aside/AsideIntro.vue'
import AsideAlbumList from '~/components/Aside/AsideAlbumList.vue'
import TrackList from '~/components/Track/TrackList.vue'
import AppPagination from '~/components/AppPagination.vue'

export default {
  components: {
    AppMainAsideWrapper,
    AppDiv,
    AppH1,
    Info,
    DivHeader,
    AsideIntro,
    AsideAlbumList,
    TrackList,
    AppPagination
  },
  computed: {
    ...mapState({
      album: state => state.album.info,
      tracks: state => state.tracks
    }),
    brief() {
      return sanitizeHtml(_.get(this.album, ['brief', 'html'], ''), {
        // TODO: refacor this options to constants
        allowedTags: ['div', 'p', 'a']
      })
    }
  },
  fetch({ store, route }) {
    const routeParam = route.params.name
    return store
      .dispatch('album/FETCH_ALBUM', {
        where: {
          name: {
            $in: [routeParam]
          }
        }
      })
      .then(() => {
        const albumId = _.get(store.state.album, ['info', 'id'], '')
        return store.dispatch('tracks/FETCH', {
          max_results: 10,
          page: 1,
          sort: '-publishedDate',
          where: {
            albums: {
              $in: [albumId]
            }
          }
        })
      })
  },
  mounted() {
    console.log(this.$sanitizeHTML)
  }
}
</script>

<style lang="stylus" scoped>
.main
  &__wrapper
    & + &
      margin 20px 0 0 0

.aside
  &__wrapper
    & + &
      margin 20px 0 0 0

.body-wrapper
  &__body
    margin 17px 0 0 0
    color #7d7d7d
    font-size 14px
    line-height 1.71
    text-align justify
    & >>> *
      margin 20px 0 0 0

.sorts
  &__sort
    border none
    background-color transparent
    padding 0 14px
    line-height 1
    font-size 14px
    color #7d7d7d
    cursor pointer
    &:focus
      outline none
    &:first-child
      padding 0 14px 0 0
    &:last-child
      padding 0 0 0 14px
    & + &
      border-left 2px solid #7d7d7d

.tracks-wrapper
  &__tracks
    margin 18px 0 0 0
  &__pagination
    margin 20px 0 0 0
</style>