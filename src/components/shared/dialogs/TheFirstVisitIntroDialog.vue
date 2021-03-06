<template>
  <v-dialog
    v-model="isFirstVisitIntroDialogVisible"
    transition="dialog-bottom-transition"
    :fullscreen="$vuetify.breakpoint.xsOnly"
    :hide-overlay="$vuetify.breakpoint.xsOnly"
    :width="$vuetify.breakpoint.xsOnly ? undefined : 600"
  >
    <v-card>
      <v-window v-model="itemNum">
        <v-window-item v-for="item in windowItems" :key="item.id">
          <v-card flat color="transparent">
            <v-img
              :src="item.imgSrc"
              :lazy-src="item.imgLazySrc"
              aspect-ratio="1.6"
              contain
              :width="$vuetify.breakpoint.xsOnly ? undefined : 600"
            >
              <template v-slot:placeholder>
                <v-row align="center" justify="center" class="ma-0">
                  <v-progress-circular
                    indeterminate
                    color="grey lighten-5"
                  ></v-progress-circular>
                </v-row>
              </template>
            </v-img>
            <v-card-text>
              <span class="blue-grey--text">{{ item.body }}</span>
            </v-card-text>
          </v-card>
        </v-window-item>
      </v-window>
      <v-card-actions class="justify-center">
        <v-item-group v-model="itemNum" mandatory>
          <v-item v-for="item in windowItems" :key="item.id">
            <v-btn
              aria-label="Next"
              small
              class="blue-grey--text"
              slot-scope="{ active, toggle }"
              :input-value="active"
              icon
              @click="toggle"
            >
              <v-icon>mdi-record</v-icon>
            </v-btn>
          </v-item>
        </v-item-group>
      </v-card-actions>
      <v-card-actions class="justify-center">
        <v-btn
          aria-label="Next"
          v-if="itemNum + 1 !== itemCount"
          color="primary"
          class="mono"
          text
          @click.stop="next"
        >
          {{ $t('intro.nextButtonText') }}
        </v-btn>
        <v-btn
          aria-label="End"
          v-else
          color="primary"
          class="mono"
          @click.stop="onEndIntroClick"
        >
          {{ $t('intro.closeButtonText') }}
        </v-btn>
      </v-card-actions>
    </v-card>
  </v-dialog>
</template>

<script>
import { mapActions } from 'vuex'
import { siteImagesBaseUrl } from '@/utils/constants'

export default {
  name: 'TheFirstVisiIntroDialog',
  data() {
    return {
      isFirstVisitIntroDialogVisible: true,
      itemNum: 0,
      windowItems: [
        {
          id: 1,
          imgSrc: `${siteImagesBaseUrl}/webp/intro_doodle_1.webp`,
          imgLazySrc: `${siteImagesBaseUrl}/webp/lazy/intro_doodle_1.webp`,
          title: this.$i18n.t('intro.item1.title'),
          body: this.$i18n.t('intro.item1.body')
        },
        {
          id: 2,
          imgSrc: `${siteImagesBaseUrl}/webp/intro_doodle_2.webp`,
          imgLazySrc: `${siteImagesBaseUrl}/webp/lazy/intro_doodle_2.webp`,
          title: this.$i18n.t('intro.item2.title'),
          body: this.$i18n.t('intro.item2.body')
        },
        {
          id: 3,
          imgSrc: `${siteImagesBaseUrl}/webp/intro_doodle_3.webp`,
          imgLazySrc: `${siteImagesBaseUrl}/webp/lazy/intro_doodle_3.webp`,
          title: this.$i18n.t('intro.item3.title'),
          body: this.$i18n.t('intro.item3.body')
        }
      ]
    }
  },
  computed: {
    itemCount() {
      return this.windowItems.length
    }
  },
  methods: {
    ...mapActions({
      setHasVisitedBefore: 'setHasVisitedBefore'
    }),
    next() {
      this.itemNum = this.itemNum + 1 === this.itemCount ? 0 : this.itemNum + 1
    },
    prev() {
      this.itemNum =
        this.itemNum - 1 < 0 ? this.itemCount - 1 : this.itemNum - 1
    },
    onEndIntroClick() {
      // TESTME: For testing welcome screen, remove following line as well as the existing localStorage object in browser
      this.setHasVisitedBefore('true')
      this.isFirstVisitIntroDialogVisible = false
    }
  }
}
</script>

<style scoped>
.mono {
  font-family: monospace;
}
</style>
