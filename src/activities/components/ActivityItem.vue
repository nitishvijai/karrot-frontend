<template>
  <QCard
    :class="{ full: activity.isFull }"
  >
    <QCardSection
      class="no-padding content"
      :class="{ isEmpty: activity.isEmpty, isUserMember: activity.isUserMember, isDisabled: activity.isDisabled }"
    >
      <div class="content-inner">
        <div class="row no-wrap items-start justify-between">
          <div>
            <strong class="featured-text">
              {{ $d(activity.date, 'hourMinute') }}
              <template v-if="activity.hasDuration"> &mdash; {{ $d(activity.dateEnd, 'hourMinute') }}</template>
            </strong>
            <template v-if="placeLink">
              <template v-if="dense">
                <span>{{ $d(activity.date, 'dateWithDayName') }}</span>
                <br>
              </template>
              <span v-if="activity.place">
                <RouterLink :to="{ name: 'place', params: { placeId: activity.place.id }}">
                  {{ activity.place.name }}
                </RouterLink>
              </span>
            </template>
            <template v-else>
              {{ $d(activity.date, 'dateLongWithDayName') }}
            </template>
          </div>
          <QIcon
            v-if="activity.activityType"
            v-bind="activity.activityType.iconProps"
            title=""
            size="xs"
            class="q-ml-sm"
            style="top: 1px;"
          >
            <q-tooltip v-if="activity.activityType.iconProps && activity.activityType.iconProps.title">
              {{ activity.activityType.iconProps.title }}
            </q-tooltip>
          </QIcon>
        </div>
        <div
          v-if="activity.isDisabled"
          class="q-my-xs"
        >
          <b class="text-negative">{{ $t('ACTIVITYLIST.ACTIVITY_DISABLED') }}</b>
        </div>
        <div
          v-if="activity.hasStarted"
          class="q-my-xs"
        >
          <b class="text-orange">{{ $t('ACTIVITYLIST.ACTIVITY_STARTED') }}</b>
        </div>
        <!-- eslint-disable vue/multiline-html-element-content-newline -->
        <div
          v-if="activity.description"
          class="q-my-xs multiline"
        >{{ activity.description }}</div>
        <!-- eslint-enable vue/multiline-html-element-content-newline -->
        <div class="q-mt-sm q-mb-none full-width">
          <div
            v-for="participantType in participantTypes"
            :key="participantType.id"
          >
            <template v-if="showRoleInfo">
              <template v-if="participantType.name">
                <strong>{{ participantType.name }}</strong> ({{ participantType.role }})
              </template>
              <strong v-else>
                {{ participantType.role }}
              </strong>
              <template v-if="roles.includes(participantType.role)">
                <div>{{ participantType.description }}</div>
              </template>
            </template>
            <ActivityUsers
              :activity="activity"
              :participant-type="participantType"
              @leave="leave"
              @join="join"
            />
          </div>
          <CustomDialog v-model="joinDialog">
            <template #title>
              <QIcon
                v-bind="activity.activityType.iconProps"
                size="sm"
                class="q-pr-sm"
              />
              {{ $t('ACTIVITYLIST.ITEM.JOIN_CONFIRMATION_HEADER', { activityType: activity.activityType.translatedName }) }}
            </template>
            <template #message>
              <template v-if="showAvailableRoleInfo">
                <QItem
                  v-for="participantType in availableparticipantTypes"
                  :key="participantType.role"
                  tag="label"
                >
                  <QItemSection avatar>
                    <QRadio
                      v-model="joinParticipantTypeId"
                      :val="participantType.id"
                      color="orange"
                    />
                  </QItemSection>
                  <QItemSection>
                    <QItemLabel>
                      <template v-if="participantType.name">
                        <strong>{{ participantType.name }}</strong><br>
                        {{ participantType.role }} role
                      </template>
                      <strong v-else>
                        {{ participantType.role }} role
                      </strong>
                    </QItemLabel>
                    <QItemLabel caption>
                      {{ participantType.description }}
                    </QItemLabel>
                  </QItemSection>
                </QItem>
                <br>
              </template>
              {{ $t('ACTIVITYLIST.ITEM.JOIN_CONFIRMATION_TEXT', { date: $d(activity.date, 'long') }) }}
            </template>
            <template #actions>
              <QBtn
                v-close-popup
                flat
                color="primary"
                :label="$t('BUTTON.CANCEL')"
              />
              <QBtn
                v-close-popup
                flat
                color="primary"
                data-autofocus
                :label="$t('BUTTON.OF_COURSE')"
                @click="$emit('join', { activityId: activity.id, participantTypeId: joinParticipantTypeId } )"
              />
            </template>
          </CustomDialog>
          <CustomDialog v-model="leaveDialog">
            <template #title>
              <QIcon
                v-bind="activity.activityType.iconProps"
                size="sm"
                class="q-pr-sm"
              />
              {{ $t('ACTIVITYLIST.ITEM.LEAVE_CONFIRMATION_HEADER', { activityType: activity.activityType.translatedName }) }}
            </template>
            <template #message>
              {{ $t('ACTIVITYLIST.ITEM.LEAVE_CONFIRMATION_TEXT') }}
            </template>
            <template #actions>
              <QBtn
                v-close-popup
                flat
                color="primary"
                :label="$t('BUTTON.CANCEL')"
              />
              <QBtn
                v-close-popup
                flat
                color="primary"
                data-autofocus
                :label="$t('BUTTON.YES')"
                @click="$emit('leave', activity.id)"
              />
            </template>
          </CustomDialog>
        </div>
      </div>
    </QCardSection>
    <QCardSection
      class="row no-padding full-width justify-end bottom-actions"
    >
      <QBtn
        v-if="hasJoined"
        flat
        no-caps
        color="negative"
        class="action-button"
        @click="leave"
      >
        <QIcon
          name="fas fa-user-times"
          size="xs"
          class="q-mr-sm"
        />
        <span class="block">Leave {{ activity.activityType.name }}</span>
      </QBtn>
      <QBtn
        v-else
        flat
        no-caps
        :color="activity.activityType.iconProps.color"
        class="action-button"
        @click="join"
      >
        <QIcon
          name="fas fa-user-plus"
          size="xs"
          class="q-mr-sm"
        />
        <span class="block">Join {{ activity.activityType.name }}</span>
      </QBtn>
      <QSpace />
      <QBtn
        v-if="activity.isUserMember"
        :href="icsUrl"
        flat
        no-caps
        type="a"
        color="secondary"
        class="action-button"
      >
        <template #default>
          <QIcon
            name="event"
            size="xs"
            class="q-mr-xs"
          />
          <span>{{ $t('ACTIVITYLIST.ITEM.DOWNLOAD_ICS') }}</span>
        </template>
      </QBtn>
      <QBtn
        flat
        no-caps
        color="secondary"
        class="action-button"
        @click.native.stop="detail"
      >
        <template #default>
          <QIcon
            name="chat"
            size="xs"
            class="q-mr-xs icon-chat"
          />
          <span class="block">{{ $t('CONVERSATION.OPEN') }}</span>
        </template>
      </QBtn>
    </QCardSection>
  </QCard>
</template>

<script>
import {
  QCard,
  QCardSection,
  QIcon,
  QBtn,
  QSpace,
  QItem,
  QItemSection,
  QItemLabel,
  QRadio,
} from 'quasar'
import ActivityUsers from './ActivityUsers'
import CustomDialog from '@/utils/components/CustomDialog'
import { absoluteURL } from '@/utils/absoluteURL'

export default {
  components: {
    CustomDialog,
    QCard,
    QCardSection,
    QIcon,
    QBtn,
    QSpace,
    QItem,
    QItemSection,
    QItemLabel,
    QRadio,
    ActivityUsers,
  },
  props: {
    activity: {
      type: Object,
      required: true,
    },
    roles: {
      type: Array,
      required: true,
    },
    dense: {
      type: Boolean,
      default: false,
    },
    placeLink: {
      type: Boolean,
      default: false,
    },
  },
  data () {
    return {
      joinDialog: false,
      leaveDialog: false,
      joinParticipantTypeId: null,
    }
  },
  computed: {
    hasJoined () {
      return this.activity.participants.find(participant => participant.user.isCurrentUser)
    },
    participantTypes () {
      return this.activity.participantTypes.filter(entry => !entry._removed)
    },
    showRoleInfo () {
      return this.participantTypes.length > 1 || this.participantTypes[0].name || this.participantTypes[0].description
    },
    availableparticipantTypes () {
      return this.participantTypes.filter(participantType => {
        return this.roles.includes(participantType.role) && !participantType.isFull
      })
    },
    showAvailableRoleInfo () {
      return this.availableparticipantTypes.length > 1 || this.availableparticipantTypes[0].name || this.availableparticipantTypes[0].description
    },
    icsUrl () {
      // a relative URL would work fine in a browser but
      // not in Cordova so we always make it absolute for simplicity.
      // see https://github.com/karrot-dev/karrot-frontend/issues/2400
      return absoluteURL(`/api/activities/${this.activity.id}/ics/`)
    },
  },
  methods: {
    join () {
      this.joinParticipantTypeId = this.availableparticipantTypes.length > 0 && this.availableparticipantTypes[0].id
      this.leaveDialog = false
      this.joinDialog = true
    },
    leave () {
      if (!this.activity.hasStarted) {
        this.joinDialog = false
        this.leaveDialog = true
      }
    },
    detail (event) {
      if (event.target.closest('a')) return // ignore actual links
      this.$emit('detail', this.activity)
    },
  },
}
</script>

<style scoped lang="stylus">
@import '~variables'

.content
  width 100%
  transition background-color 2s ease

  &.isUserMember
    &:not(.isDisabled)
      background linear-gradient(to right, $lightGreen, $lighterGreen)

  &.isDisabled
    background $lightRed

  .content-inner
    width 100%
    padding 12px

    .featured-text
      display inline
      margin-right .3em

.bottom-actions
  font-weight 500
  color $secondary
  box-shadow 0 -1px 0 rgba(0, 0, 0, 0.06)

.q-btn.action-button
  >>> .q-btn__wrapper
    padding 10px 16px !important

    .icon-chat
      transform rotateY(180deg)
</style>
