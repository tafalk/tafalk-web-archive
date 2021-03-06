<template>
  <v-dialog
    v-model="getIsStopWatchingConfirmationDialogVisible"
    persistent
    max-width="290"
  >
    <v-card>
      <v-card-title class="headline">{{
        $t('user.stopWatch.dialog.title')
      }}</v-card-title>
      <v-card-text>{{ $t('user.stopWatch.dialog.body') }}</v-card-text>
      <v-card-actions>
        <v-spacer></v-spacer>
        <v-btn
          aria-label="Yes"
          color="red darken-1"
          text
          @click.native="onStopWatchingThisUserConfirmClick"
          >{{ $t('common.options.yesButtonText') }}</v-btn
        >
        <v-btn
          aria-label="No"
          color="light-blue darken-1"
          text
          @click.native="setIsStopWatchingConfirmationDialogVisible(false)"
          >{{ $t('common.options.noButtonText') }}</v-btn
        >
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import API, { graphqlOperation } from '@aws-amplify/api'
import { mapGetters, mapActions, mapMutations } from 'vuex'
import { StopWatchingUser } from '@/graphql/UserInteraction'

export default {
  name: 'StopWatchingConfirmationDialog',
  data() {
    return {
      watchTypeUserConnectionValue: 'Watch'
    }
  },
  computed: {
    ...mapGetters({
      getIsStopWatchingConfirmationDialogVisible:
        'visitedUser/dialog/getIsStopWatchingConfirmationDialogVisible',
      getAuthenticatedUser: 'authenticatedUser/getUser',
      getVisitedUser: 'visitedUser/getUser'
    }),
    authenticatedUser() {
      return this.getAuthenticatedUser
    },
    visitedUser() {
      return this.getVisitedUser
    }
  },
  methods: {
    ...mapMutations({
      setIsStopWatchingConfirmationDialogVisible:
        'visitedUser/dialog/setIsStopWatchingConfirmationDialogVisible',
      clearInboundWatchIdFromAuthenticatedUser:
        'visitedUser/clearInboundWatchIdFromAuthenticatedUser'
    }),
    ...mapActions({
      setNewSiteError: 'shared/setNewSiteError'
    }),
    async onStopWatchingThisUserConfirmClick() {
      try {
        const watchId = this.visitedUser.connectionsWithAuthenticatedUser
          .inbound.watchId

        if (watchId && watchId.length > 0) {
          await API.graphql(
            graphqlOperation(StopWatchingUser, {
              watchId
            })
          )
          this.clearInboundWatchIdFromAuthenticatedUser()
        } else {
          throw new Error(
            this.$i18n.t('user.stopWatch.message.notWatchedError')
          )
        }
      } catch (err) {
        this.setNewSiteError(err.message ?? err)
      } finally {
        // close the dialog
        this.setIsStopWatchingConfirmationDialogVisible(false)
      }
    }
  }
}
</script>
