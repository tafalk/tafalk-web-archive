<template>
  <v-dialog
    v-model="getIsLanguageChooseDialogVisible"
    persistent
    max-width="750px"
  >
    <v-card>
      <v-card-title class="headline">{{
        $t('siteLanguage.edit.dialog.title')
      }}</v-card-title>
      <v-card-text>
        <p>{{ $t('siteLanguage.edit.dialog.body') }}</p>

        <v-select
          v-model="languageModel"
          dense
          flat
          :label="$t('siteLanguage.edit.dialog.selectLabel')"
          :items="languageOptions"
          return-object
        ></v-select>

        <v-alert :value="unsupportedLanguageAlert" type="info">
          {{ unsupportedLanguageAlert }}
        </v-alert>
      </v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          aria-label="Close"
          color="blue darken-1"
          text
          @click.native="setIsLanguageChooseDialogVisible(false)"
          >{{ $t('common.options.closeButtonText') }}</v-btn
        >
        <v-btn
          aria-label="Save"
          color="blue darken-1"
          text
          @click.native="onSaveLanguageEditClick"
          >{{ $t('common.options.saveButtonText') }}</v-btn
        >
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { mapGetters, mapMutations, mapActions } from 'vuex'
import { languageOptions } from '@/utils/constants'

export default {
  name: 'UserLanguageEditDialog',
  data() {
    return {
      languageModel: null,
      languageOptions
    }
  },
  created() {
    // Set initial value for current user's preferred language
    if (!this.getAuthenticatedUser?.language) {
      this.languageModel = languageOptions.find(o => o.value === '')
      return
    }
    this.languageModel = languageOptions.find(
      o => o.value === this.authenticatedUser.language
    )
  },
  computed: {
    ...mapGetters({
      getAuthenticatedUser: 'authenticatedUser/getUser',
      getIsLanguageChooseDialogVisible:
        'authenticatedUser/dialog/getIsLanguageChooseDialogVisible'
    }),
    authenticatedUser() {
      return this.getAuthenticatedUser ?? {}
    },
    unsupportedLanguageAlert() {
      // If one of non-default values have chosen, no need to check
      if (this.languageModel.value) return null

      const browserLanguage = navigator.language.split('-')[0]
      // Browser language is supported.
      if (this.languageOptions.some(o => o.value === browserLanguage))
        return null
      return this.$i18n.t('siteLanguage.edit.dialog.unsupportedAlertText', {
        browserLanguage
      })
    }
  },
  methods: {
    ...mapMutations({
      setIsLanguageChooseDialogVisible:
        'authenticatedUser/dialog/setIsLanguageChooseDialogVisible'
    }),
    ...mapActions({
      setLanguage: 'authenticatedUser/setLanguage',
      setNewSiteError: 'shared/setNewSiteError'
    }),
    async onSaveLanguageEditClick() {
      try {
        const langCode = this.languageModel.value
          ? this.languageModel.value
          : ''

        await this.setLanguage({
          userId: this.authenticatedUser.id,
          language: langCode
        })
        const browserLanguage = navigator.language.split('-')[0]
        this.$i18n.locale = langCode ?? browserLanguage
      } catch (err) {
        this.setNewSiteError(err.message ?? err)
      } finally {
        this.setIsLanguageChooseDialogVisible(false)
      }
    }
  }
}
</script>
