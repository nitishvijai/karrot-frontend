<template>
  <QCard
    v-if="group"
    class="no-mobile-margin no-shadow grey-border"
  >
    <RandomArt
      :seed="group.id"
      type="circles"
    >
      <div class="art-overlay q-px-md q-py-lg">
        <div
          v-t="'GROUP.DESCRIPTION'"
          class="header"
        />
      </div>
    </RandomArt>
    <div class="q-mx-md actionButtons">
      <RouterLink
        v-if="isEditor"
        :to="{name: 'groupEdit'}"
      >
        <QBtn
          small
          round
          color="secondary"
          icon="fas fa-pencil-alt"
          :title="$t('GROUP.EDIT')"
        />
      </RouterLink>
    </div>
    <div class="q-pa-md">
      <Markdown
        v-if="group.description"
        :source="group.description"
        mentions
      />
    </div>
  </QCard>
</template>

<script>
import {
  QCard,
  QBtn,
} from 'quasar'
import Markdown from '@/utils/components/Markdown'
import RandomArt from '@/utils/components/RandomArt'

import {
  mapGetters,
} from 'vuex'

export default {
  components: {
    QCard,
    QBtn,
    RandomArt,
    Markdown,
  },
  computed: {
    ...mapGetters({
      group: 'currentGroup/value',
      isEditor: 'currentGroup/isEditor',
    }),
  },
}
</script>

<style scoped lang="stylus">
@import '~variables'

.actionButtons
  float right
  margin-top -21px

  .q-btn
    margin-left 3px

.art-overlay
  color white
  background rgba(0, 0, 0, 0)

  .header
    font-size 1.3em

  .subtitle
    margin-top 6px

body.desktop .art-overlay
  background linear-gradient(to top, rgba(0, 0, 0, 0.5) 0%, rgba(0, 0, 0, 0) 58%, rgba(0, 0, 0, 0) 90%)
</style>
