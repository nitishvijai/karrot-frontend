<template>
  <div>
    <QCard
      v-for="(participantType, idx) in visibleParticipantTypes"
      :key="participantType.id || `new-${idx}`"
      flat
    >
      <QCardSection>
        <div class="text-h6">
          Participant type {{ idx + 1 }}
        </div>
      </QCardSection>
      <QCardSection>
        <MarkdownInput
          v-model="participantType.description"
          :error="hasError('description')"
          :error-message="firstError('description')"
          :label="$t('CREATEACTIVITY.COMMENT')"
          :hint="$t('CREATEACTIVITY.COMMENT_HELPER')"
          icon="info"
          maxlength="500"
          input-style="min-height: auto;"
          outlined
          @keyup.ctrl.enter="maybeSave"
        />
        <QInput
          v-model.number="participantType.maxParticipants"
          type="number"
          stack-label
          outlined
          class="q-my-md"
          :label="$t('CREATEACTIVITY.MAX_PARTICIPANTS')"
          :hint="$t('CREATEACTIVITY.MAX_PARTICIPANTS_HELPER')"
          :placeholder="$t('CREATEACTIVITY.UNLIMITED')"
          :error="hasError('maxParticipants')"
          :error-message="firstError('maxParticipants')"
          input-style="max-width: 100px"
        >
          <template #before>
            <QIcon name="group" />
          </template>
          <QSlider
            v-if="participantType.maxParticipants > 0 && participantType.maxParticipants <= 10"
            v-model="participantType.maxParticipants"
            :min="1"
            :max="10"
            label
            markers
            class="q-mx-sm self-end"
            style="min-width: 60px"
          />
          <!--
          <template #after>
            <QIcon
              v-if="series ? series.maxParticipants !== edit.maxParticipants : false"
              name="undo"
              @click="edit.maxParticipants = series.maxParticipants"
            />
          </template>
          -->
        </QInput>
        <div
          v-if="seriesMeta.isMaxParticipantsChanged"
          class="q-ml-lg col-12 q-field__bottom text-warning"
        >
          <QIcon name="warning" />
          {{ $t('CREATEACTIVITY.DIFFERS_WARNING') }}
        </div>
        <QSelect
          v-model="participantType.role"
          map-options
          emit-value
          label="Open for"
          :options="roleOptions"
          :error="hasError('role')"
          :error-message="firstError('role')"
          outlined
          :behavior="smallScreen ? 'dialog' : 'menu'"
        >
          <template #before>
            <QIcon name="fas fa-key" />
          </template>
          <template #option="{ itemProps, itemEvents, opt: { label, description } }">
            <QItem
              v-bind="itemProps"
              v-on="itemEvents"
            >
              <QItemSection>
                <QItemLabel>
                  {{ label }}
                </QItemLabel>
                <QItemLabel caption>
                  {{ description }}
                </QItemLabel>
              </QItemSection>
            </QItem>
          </template>
        </QSelect>
        <div class="row justify-end">
          <QBtn
            label="Remove participant type"
            :disable="visibleParticipantTypes.length === 1"
            @click="removeSlots(participantType)"
          />
        </div>
      </QCardSection>
    </QCard>

    <div
      class="row justify-end"
    >
      <QBtn
        color="positive"
        label="Add participant type"
        @click="addSlots()"
      />
    </div>
  </div>
</template>
<script>
import {
  QCard,
  QCardSection,
  QCardActions,
  QDate,
  QTime,
  QSlider,
  QInput,
  QBtn,
  QIcon,
  QMenu,
  QDialog,
  QToggle,
  QSelect,
  QSpace,
  Dialog,
  date,
  QItem,
  QItemSection,
  QItemLabel,
} from 'quasar'

import MarkdownInput from '@/utils/components/MarkdownInput'
import editMixin from '@/utils/mixins/editMixin'
import statusMixin from '@/utils/mixins/statusMixin'

export default {
  components: {
    QDate,
    QTime,
    QSlider,
    QInput,
    QBtn,
    QIcon,
    QMenu,
    QDialog,
    QToggle,
    QSelect,
    QCard,
    QCardSection,
    QCardActions,
    QSpace,
    MarkdownInput,
    QItem,
    QItemSection,
    QItemLabel,
  },
  mixins: [statusMixin],
  props: {
    value: {
      required: true,
      type: Array,
    },
    seriesMeta: {
      required: true,
      type: Object,
    },
    roles: {
      type: Array,
      required: true,
    },
  },
  data () {
    return {
      participantTypes: [],
    }
  },
  computed: {
    visibleParticipantTypes () {
      return this.participantTypes.filter(entry => !entry._removed)
    },
    roleOptions () {
      return [
        {
          label: 'Anyone',
          value: 'member',
          description: 'Anyone in the group',
        },
        {
          label: 'Newcomer',
          value: 'newcomer',
          description: 'People that haven\'t yet got any other roles',
        },
        {
          label: 'Approved',
          value: 'approved',
          description: 'People that have been trusted with approved role',
        },
        {
          label: 'Editor',
          value: 'editor',
          description: 'People that have been trusted in the group as an editor',
        },
      ]
    },
    smallScreen () {
      return this.$q.screen.width < 450 || this.$q.screen.height < 450
    },
  },
  watch: {
    value: {
      handler (val) {
        this.participantTypes = val
      },
      immediate: true,
    },
  },
  methods: {
    addSlots () {
      this.participantTypes.push({
        role: this.roles[0],
        maxParticipants: 2,
      })
    },
    removeSlots (participantType) {
      if (participantType.id) {
        this.$set(participantType, '_removed', true)
      }
      else {
        const idx = this.participantTypes.indexOf(participantType)
        this.participantTypes.splice(idx, 1)
      }
    },
  },
}
</script>
