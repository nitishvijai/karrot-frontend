<template>
  <div
    class="bg-white relative-position"
  >
    <KSpinner v-show="fetchInitialPending" />
    <template v-if="!fetchInitialPending">
      <QList>
        <QItem
          v-if="conversations.length === 0"
        >
          {{ $t('CONVERSATION.NO_CONVERSATIONS') }}
        </QItem>
        <LatestMessageItem
          v-for="conv in conversations"
          :key="conv.id"
          v-close-popup
          :group="conv.type === 'group' ? conv.target : null"
          :user="conv.type === 'private' ? conv.target : null"
          :activity="conv.type === 'activity' ? conv.target : null"
          :place="conv.type === 'place' ? conv.target : null"
          :application="conv.type === 'application' ? conv.target : null"
          :issue="conv.type === 'issue' ? conv.target : null"
          :offer="conv.type === 'offer' ? conv.target : null"
          :message="conv.latestMessage"
          :unread-count="conv.unreadMessageCount"
          :muted="conv.muted"
          :closed="conv.isClosed"
          :selected="isSelected(conv)"
          @open="open(conv)"
        />
        <QItem
          v-if="!asPopover && canFetchPastConversations"
          class="row justify-center"
        >
          <QBtn
            size="sm"
            :loading="fetchingPastConversations"
            @click="fetchPastConversations"
          >
            {{ $t('BUTTON.SHOW_MORE') }}
          </QBtn>
        </QItem>
        <div
          v-if="asPopover"
          class="row justify-end q-my-sm q-mr-sm"
        >
          <QBtn
            v-close-popup
            size="sm"
            color="secondary"
            :to="{ name: 'latestConversations' }"
          >
            {{ $t('BUTTON.SHOW_MORE') }}
          </QBtn>
        </div>
      </QList>
    </template>
  </div>
</template>

<script>
import {
  QList,
  QItem,
  QBtn,
} from 'quasar'
import { mapGetters, mapActions } from 'vuex'
import LatestMessageItem from './LatestMessageItem'
import KSpinner from '@/utils/components/KSpinner'

export default {
  components: {
    QList,
    QItem,
    QBtn,
    LatestMessageItem,
    KSpinner,
  },
  props: {
    asPopover: {
      type: Boolean,
      default: false,
    },
  },
  computed: {
    ...mapGetters({
      conversations: 'latestMessages/conversations',
      canFetchPastConversations: 'latestMessages/canFetchPastConversations',
      fetchingPastConversations: 'latestMessages/fetchingPastConversations',
      fetchInitialPending: 'latestMessages/fetchInitialPending',
      selectedConversation: 'detail/conversation',
    }),
  },
  mounted () {
    this.fetchInitial()
    setTimeout(() => this.markConversationsSeen(), 3 * 1000)
  },
  destroyed () {
    this.markConversationsSeen()
  },
  methods: {
    ...mapActions({
      openForActivity: 'detail/openForActivity',
      openForUser: 'detail/openForUser',
      openForApplication: 'detail/openForApplication',
      fetchPastConversations: 'latestMessages/fetchPastConversations',
      fetchInitial: 'latestMessages/fetchInitial',
      markConversationsSeen: 'latestMessages/markConversationsSeen',
    }),
    open (conv) {
      const { type, target } = conv
      switch (type) {
        case 'group': return this.$router.push({ name: 'group', params: { groupId: target.id }, hash: '#messages' }).catch(() => {})
        case 'place': return this.$router.push({ name: 'placeWall', params: { groupId: target.group.id, placeId: target.id } }).catch(() => {})
        case 'activity': return this.openForActivity(target)
        case 'private': return this.openForUser(target)
        case 'application': return this.openForApplication(target)
        case 'issue': return this.$router.push({ name: 'issueChat', params: { groupId: target.group.id, issueId: target.id } }).catch(() => {})
        case 'offer': return this.$router.push({
          name: 'offerDetail',
          params: { groupId: target.group.id, offerId: target.id },
          query: this.$route.query,
        }).catch(() => {})
      }
    },
    isSelected (conv) {
      if (!this.selectedConversation) return false
      if (Boolean(conv.thread) !== Boolean(this.selectedConversation.thread)) return false
      return conv.id === this.selectedConversation.id
    },
  },
}
</script>
