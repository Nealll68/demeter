<template>
  <v-bottom-sheet v-model="show" @click:outside="$emit('close')">
    <v-sheet color="grey darken-4">
      <v-container class="app-container">
        <v-card color="transparent" flat>
          <v-card-title class="pb-0">
            <v-row :no-gutters="$vuetify.breakpoint.smAndDown">
              <v-col
                cols="12"
                md="6"
              >
                <v-text-field
                  v-model="workshopId"
                  filled
                  :label="$t('workshop.idWorkshop')"
                  prepend-inner-icon="mdi-plus-box"
                ></v-text-field>
              </v-col>

              <v-col
                cols="12"
                md="6"
              >
                <v-text-field
                  v-model="collectionId"
                  filled
                  :label="$t('workshop.idCollection')"
                  prepend-inner-icon="mdi-plus-box-multiple"
                ></v-text-field>
              </v-col>
            </v-row>
          </v-card-title>

          <v-card-text v-if="getDetailsLoading" class="text-center">
            <v-progress-circular
              indeterminate
              color="primary"
            ></v-progress-circular>
          </v-card-text>

          <v-card-text
            v-if="modsDetails.length > 0"
            style="max-height: 300px"
            class="scrollbar"
          >
            <v-list
              v-for="mod of modsDetails"
              :key="mod.publishedfileid"
              color="transparent"
            >
              <v-list-item>
                <v-list-item-avatar>
                  <v-img :src="mod.preview_url"></v-img>
                </v-list-item-avatar>

                <v-list-item-content>
                  <v-list-item-title>{{ mod.title }}</v-list-item-title>
                  <v-list-item-subtitle
                    >{{ $t('workshop.publishedAt') }}
                    {{ 
                      $moment
                        .unix(mod.time_created)
                        .format('ll')
                    }}</v-list-item-subtitle
                  >
                  <v-list-item-subtitle
                    >{{ $t('common.lastUpdate') }} :
                    {{
                      $moment
                        .unix(mod.time_updated)
                        .format('lll')
                    }}</v-list-item-subtitle
                  >
                  <v-list-item-subtitle
                    >{{ $t('common.size') }} :
                    {{ mod.file_size | formatBytes }}</v-list-item-subtitle
                  >
                </v-list-item-content>

                <v-list-item-action>
                  <v-btn
                    color="quaternary"
                    text
                    @click="
                      downloadInfo(
                        mod.publishedfileid,
                        mod.title,
                        mod.file_size,
                        mod.file_url,
                        mod.filename,
                        mod.preview_url
                      )
                    "
                    :disabled="$store.state.downloadInfo.type ? true : false"
                  >
                    <v-icon left>mdi-download</v-icon>{{ $t('common.download') }}
                  </v-btn>
                </v-list-item-action>
              </v-list-item>

              <v-divider inset></v-divider>
            </v-list>
          </v-card-text>
        </v-card>
      </v-container>
    </v-sheet>
  </v-bottom-sheet>
</template>

<script>
import debounce from 'debounce'

export default {
  props: ['show'],

  data: () => ({
    getDetailsLoading: false,
    workshopId: null,
    collectionId: null,
    modsDetails: []
  }),

  watch: {
    workshopId() {
      this.getWorkshopDetails()
    },

    collectionId() {
      this.getCollectionDetails()
    }
  },

  methods: {
    extractIdFromUrl (url) {
      if (url.search('https://steamcommunity.com/sharedfiles/filedetails') !== -1) {
        return url.substring(url.search('id=') + 3, url.length)
      }

      return url
    },

    downloadInfo(workshopId, title, fileSize, fileUrl, filename, image) {
      this.$emit('download-info', {
        workshopId,
        title,
        fileSize,
        fileUrl,
        filename,
        image
      })
    },

    getWorkshopDetails: debounce(async function() {
      this.getDetailsLoading = true

      if (this.workshopId) {
        try {
          this.collectionId = null
          this.modsDetails = []

          const response = await this.$axios.$get(`server/workshop/file/${this.extractIdFromUrl(this.workshopId)}`)
          this.modsDetails.push(response)
        } catch (ex) {
          this.$snackbar({
            type: 'error',
            message: this.$t('workshop.idError')
          })
        }
      }

      this.getDetailsLoading = false
    }, 500),

    getCollectionDetails: debounce(async function() {
      this.getDetailsLoading = true

      if (this.collectionId) {
        try {
          this.workshopId = null
          this.modsDetails = []

          const response = await this.$axios.$get(`server/workshop/collection/${this.extractIdFromUrl(this.collectionId)}`)
          this.modsDetails = response
        } catch (ex) {
          this.$snackbar({
            type: 'error',
            message: this.$t('workshop.idError')
          })
        }
      }

      this.getDetailsLoading = false
    }, 500)
  }
}
</script>
