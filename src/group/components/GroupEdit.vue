<template>
  <component :is="$q.platform.is.mobile ? 'div' : 'QCard'">
    <ChangePhoto
      v-if="!isNew"
      :hint="$t('GROUP.SET_LOGO')"
      :label="$t('GROUP.LOGO')"
      :status="status"
      :value="value"
      @save="$emit('save', { id: value.id, photo: arguments[0] })"
    />
    <div
      class="edit-box"
      :class="{ changed: hasChanged }"
    >
      <form @submit.prevent="maybeSave">
        <h4 class="text-primary q-mt-none q-mb-lg">
          {{ $t('GROUP.HEADINGS.GENERAL') }}
        </h4>

        <QInput
          v-if="!edit.isPlayground"
          id="group-title"
          v-model="edit.name"
          :autofocus="!$q.platform.has.touch"
          :error="hasNameError"
          :error-message="nameError"
          :label="$t('GROUP.TITLE')"
          autocomplete="off"
          outlined
          class="q-mb-lg"
          @blur="$v.edit.name.$touch"
        >
          <template #before>
            <QIcon name="fas fa-fw fa-star" />
          </template>
        </QInput>

        <MarkdownInput
          v-if="!edit.isPlayground"
          v-model="edit.publicDescription"
          :error="hasError('publicDescription')"
          :error-message="firstError('publicDescription')"
          :label="$t('GROUPINFO.TITLE')"
          icon="fas fa-fw fa-question"
          outlined
          class="q-mb-lg"
          @keyup.ctrl.enter="maybeSave"
        />

        <MarkdownInput
          v-model="edit.description"
          :error="hasError('description')"
          :error-message="firstError('description')"
          :label="$t('GROUP.DESCRIPTION_VERBOSE')"
          icon="fas fa-fw fa-address-card"
          outlined
          mentions
          class="q-mb-lg"
          @keyup.ctrl.enter="maybeSave"
        />

        <h4 class="text-primary q-mt-xl q-mb-lg">
          {{ $t('GROUP.HEADINGS.LOCATION') }}
          <InfoPopup
            :title="$t('INFO')"
            :description="$t('GROUP.HEADINGS.LOCATION_HINT')"
          />
        </h4>

        <AddressPicker
          v-model="edit"
          :color="isNew ? 'blue' : 'positive'"
          :error="hasAddressError"
          :error-message="addressError"
          :label="$t('GROUP.ADDRESS')"
          :default-map-center="defaultMapCenter"
          font-icon="fas fa-home"
          icon="fas fa-fw fa-map-marker"
          outlined
          class="q-mb-lg"
        />

        <QSelect
          v-model="edit.timezone"
          :error="hasTimezoneError"
          :error-message="timezoneError"
          :label="$t('GROUP.TIMEZONE')"
          :options="filteredTimezones"
          fill-input
          hide-selected
          input-debounce="0"
          outlined
          use-input
          class="q-mb-lg"
          @filter="timezoneFilter"
          @blur="$v.edit.timezone.$touch"
        >
          <template #before>
            <QIcon name="fas fa-fw fa-globe" />
          </template>
        </QSelect>

        <h4 class="text-primary q-mt-xl q-mb-lg">
          {{ $t('GROUP.HEADINGS.NEW_MEMBERS_SIGNUP') }}
          <InfoPopup
            :title="$t('INFO')"
            :description="$t('GROUP.HEADINGS.NEW_MEMBERS_SIGNUP_HINT')"
          />
        </h4>

        <MarkdownInput
          v-if="!edit.isOpen"
          :error="hasError('applicationQuestions')"
          :error-message="firstError('applicationQuestions')"
          :label="$t('GROUP.APPLICATION_QUESTIONS')"
          :value="applicationQuestionsOrDefault"
          icon="fas fa-fw fa-question"
          outlined
          class="q-mb-lg"
          @input="applicationQuestionsInput"
          @keyup.ctrl.enter="maybeSave"
        />

        <MarkdownInput
          v-if="!edit.isOpen"
          v-model="edit.welcomeMessage"
          :error="hasError('welcomeMessage')"
          :error-message="firstError('welcomeMessage')"
          :label="$t('GROUP.WELCOMEMESSAGE_VERBOSE')"
          icon="fas fa-fw fa-address-card"
          outlined
          class="q-mb-lg"
          @keyup.ctrl.enter="maybeSave"
        />

        <div
          v-if="hasNonFieldError"
          class="text-negative"
        >
          {{ firstNonFieldError }}
        </div>

        <div class="row justify-end q-gutter-sm q-mt-sm">
          <QBtn
            v-if="!isNew"
            type="button"
            :disable="!hasChanged"
            @click="reset"
          >
            {{ $t('BUTTON.RESET') }}
          </QBtn>

          <QBtn
            type="submit"
            color="primary"
            :disable="!canSave"
            :loading="isPending"
          >
            {{ $t(isNew ? 'BUTTON.CREATE' : 'BUTTON.SAVE_CHANGES') }}
          </QBtn>
        </div>
      </form>
    </div>
  </component>
</template>

<script>
import jstz from 'jstimezonedetect'
import {
  QCard,
  QInput,
  QBtn,
  QMenu,
  QSelect,
  QIcon,
} from 'quasar'
import AddressPicker from '@/maps/components/AddressPicker'
import MarkdownInput from '@/utils/components/MarkdownInput'
import { validationMixin } from 'vuelidate'
import { required, minLength, maxLength } from 'vuelidate/lib/validators'
import editMixin from '@/utils/mixins/editMixin'
import statusMixin from '@/utils/mixins/statusMixin'
import ChangePhoto from '@/authuser/components/Settings/ChangePhoto'
import InfoPopup from '@/utils/components/InfoPopup'

export default {
  name: 'GroupEdit',
  components: {
    QCard,
    QInput,
    QBtn,
    QMenu,
    QSelect,
    QIcon,
    AddressPicker,
    MarkdownInput,
    ChangePhoto,
    InfoPopup,
  },
  mixins: [validationMixin, editMixin, statusMixin],
  props: {
    value: {
      type: Object,
      required: false,
      default: () => ({
        name: undefined,
        publicDescription: undefined,
        description: undefined,
        timezone: jstz.determine().name(),
        latitude: undefined,
        longitude: undefined,
        address: undefined,
        applicationQuestions: undefined,
      }),
    },
    timezones: {
      type: Array,
      required: true,
    },
    allGroups: {
      type: Array,
      required: true,
    },
    defaultMapCenter: {
      default: null,
      type: Object,
    },
  },
  data () {
    return {
      filteredTimezones: this.timezones,
      show: false,
    }
  },
  computed: {
    canSave () {
      if (this.$v.edit.$error) {
        return false
      }
      if (!this.isNew && !this.hasChanged) {
        return false
      }
      return true
    },
    hasNameError () {
      return !!this.nameError
    },
    nameError () {
      if (this.$v.edit.name.$error) {
        const m = this.$v.edit.name
        if (!m.required) return this.$t('VALIDATION.REQUIRED')
        if (!m.minLength) return this.$t('VALIDATION.MINLENGTH', { min: 4 })
        if (!m.maxLength) return this.$t('VALIDATION.MAXLENGTH', { max: 81 })
        if (!m.isUnique) return this.$t('VALIDATION.UNIQUE')
      }
      return this.firstError('name')
    },
    hasTimezoneError () {
      return !!this.timezoneError
    },
    timezoneError () {
      if (this.$v.edit.timezone.$error) {
        const m = this.$v.edit.timezone
        if (!m.required) return this.$t('VALIDATION.REQUIRED')
        if (!m.inList) return this.$t('VALIDATION.VALID_TIMEZONE')
      }
      return this.firstError('timezone')
    },
    hasAddressError () {
      return !!this.addressError
    },
    addressError () {
      for (const field of ['address', 'latitude', 'longitude']) {
        if (this.hasError(field)) return this.firstError(field)
      }
      return null
    },
    applicationQuestionsOrDefault () {
      return this.edit.applicationQuestions || this.edit.applicationQuestionsDefault
    },
  },
  watch: {
    timezones (val) {
      this.filteredTimezones = val
    },
  },
  methods: {
    maybeSave (event) {
      this.$v.edit.$touch()
      if (!this.canSave) return
      this.$v.edit.$reset()
      this.save()
    },
    timezoneFilter (terms, update, abort) {
      update(() => {
        const token = terms.toLowerCase()
        this.filteredTimezones = this.timezones.filter(item => item.toLowerCase().includes(token))
      })
    },
    applicationQuestionsInput (value) {
      this.edit.applicationQuestions = value
    },
  },
  validations: {
    edit: {
      name: {
        required,
        minLength: minLength(5),
        maxLength: maxLength(80),
        isUnique (value) {
          if (value === '' || !this.value) return true
          return this.allGroups
            .filter(e => e.id !== this.value.id)
            .findIndex(e => e.name === value) < 0
        },
      },
      timezone: {
        required,
        inList (value) {
          if (value === '') return true
          if (this.timezones && this.timezones.list) {
            return this.timezones.list.findIndex(e => e.value === value) > 0
          }
          return true
        },
      },
    },
  },
}
</script>

<style scoped lang="stylus">
@import '~editbox'
</style>
