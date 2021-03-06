<template>
  <div class="uploader">
    <!-- Menu -->
    <portal to="menu">
      <section id="uploader-methods" class="menu__items">
        <menu-item
          label="Instagram"
          :icon="['fab', 'instagram']"
          :is-link="true"
          :class="{'item--active': selectionId === 1}"
          @click="selectionId = 1" />

        <menu-item
          label="Web"
          icon="globe"
          :is-link="true"
          :class="{'item--active': selectionId === 0}"
          @click="selectionId = 0" />

        <menu-item
          label="File"
          icon="file"
          :is-link="true"
          :class="{'item--active': selectionId === 2}"
          @click="selectionId = 2" />

        <menu-item
          label="Folder"
          icon="folder-open"
          :is-link="true"
          :class="{'item--active': selectionId === 3}"
          @click="selectionId = 3" />
      </section>
    </portal>

    <!-- Global alert -->
    <div v-if="alert" class="notification">
      <h5>
        <font-awesome-icon icon="exclamation-circle" />
        NOTIFICATION
      </h5>
      <div v-html="alert" />
    </div>

    <!-- DreamTime Updater -->
    <AppNotification v-if="dreamtime.available"
                     :name="`update-dreamtime-${dreamtime.latest.tag_name}`"
                     class="cursor-pointer"
                     @click="$router.push('/wizard/dreamtime')">
      🎉 <strong>{{ $dream.name }} {{ dreamtime.latest.tag_name }}</strong> is available for download!
    </AppNotification>

    <!-- DreamPower Updater -->
    <AppNotification v-if="dreampower.available"
                     :name="`update-dreampower-${dreampower.latest.tag_name}`"
                     class="cursor-pointer"
                     @click="$router.push('/wizard/power')">
      🎉 <strong>{{ dreampower.displayName }} {{ dreampower.latest.tag_name }}</strong> is available for download!
    </AppNotification>

    <!-- Checkpoints Updater -->
    <AppNotification v-if="checkpoints.available"
                     class="notification notification--warning cursor-pointer"
                     :name="`update-checkpoints-${checkpoints.latest.tag_name}`"
                     @click="$router.push('/wizard/checkpoints')">
      🎉 <strong>{{ checkpoints.displayName }} {{ checkpoints.latest.tag_name }}</strong> is available for download!
    </AppNotification>

    <div class="uploader__methods">
      <!-- Web Address -->
      <div v-show="selectionId === 0" class="methods__content">
        <PageHeader>
          <h2 class="title">
            <span class="icon"><font-awesome-icon icon="globe" /></span>
            <span>Web</span>
          </h2>

          <h3 class="subtitle">
            Upload photos and videos from the web.
            <span v-tooltip="'Only web addresses that end in: jpg, png, gif, webm or mp4.'" class="help">
              <font-awesome-icon icon="info-circle" />
            </span>
          </h3>
        </PageHeader>

        <div class="method__body text-center">
          <input v-model="webAddress"
                 type="url"
                 class="input mb-2"
                 placeholder="https://"
                 data-private="lipsum">

          <button class="button" @click="openUrl">
            <span>Upload</span>
          </button>
        </div>
      </div>

      <!-- Instagram -->
      <div v-show="selectionId === 1" class="methods__content">
        <PageHeader>
          <h2 class="title">
            <span class="icon"><font-awesome-icon :icon="['fab', 'instagram']" /></span>
            <span>Instagram</span>
          </h2>

          <h3 class="subtitle">
            Upload photos and videos from any public Instagram profile.
          </h3>
        </PageHeader>

        <div class="method__body text-center">
          <input v-model="instagramPhoto"
                 type="url"
                 class="input mb-2"
                 placeholder="https://www.instagram.com/p/dU4fHDw-Ho/"
                 data-private="lipsum">

          <button class="button"
                  @click="openInstagramPhoto">
            <span>Upload</span>
          </button>
        </div>
      </div>

      <!-- File -->
      <div v-show="selectionId === 2" class="methods__content">
        <PageHeader>
          <h2 class="title">
            <span class="icon"><font-awesome-icon icon="file" /></span>
            <span>File</span>
          </h2>

          <h3 class="subtitle">
            Upload one or more files from your computer.
          </h3>
        </PageHeader>

        <input
          v-show="false"
          ref="photo"
          type="file"
          accept="image/jpeg, image/png, image/gif, video/mp4, video/webm"
          multiple
          @change="openFile">

        <div class="method__body text-center">
          <button class="button"
                  @click.prevent="$refs.photo.click()">
            <span>Open File(s)</span>
          </button>
        </div>
      </div>

      <!-- Folder -->
      <div v-show="selectionId === 3" class="methods__content">
        <PageHeader>
          <h2 class="title">
            <span class="icon"><font-awesome-icon icon="folder" /></span>
            <span>Folder</span>
          </h2>

          <h3 class="subtitle">
            Upload all valid photos and videos from a folder on your computer.
          </h3>
        </PageHeader>

        <div class="method__body text-center">
          <button class="button"
                  @click.prevent="openFolder">
            <span>Open folder</span>
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import {
  isEmpty, map, toNumber, startsWith,
} from 'lodash'
import { dreamtrack } from '~/modules/services'
import { dreamtime, dreampower, checkpoints } from '~/modules/updater'
import { Nudify } from '~/modules/nudify'
import { tutorial } from '~/modules'
import { UploadMixin } from '~/mixins'

const { instagram } = $provider

export default {
  mixins: [UploadMixin],

  data: () => ({
    selectionId: 1,
    webAddress: '',
    instagramPhoto: '',

    dreamtime,
    dreampower,
    checkpoints,
  }),

  computed: {
    alert() {
      return dreamtrack.get('alerts.index')
    },
  },

  watch: {
    selectionId(value) {
      localStorage.uploadSelectionId = value
    },
  },

  created() {
    this.selectionId = localStorage.uploadSelectionId || 1
    this.selectionId = toNumber(this.selectionId)
  },

  mounted() {
    tutorial.upload()
  },

  methods: {
    /**
     *
     */
    openFile(event) {
      const { files } = event.target

      if (files.length === 0) {
        return
      }

      const paths = map(files, 'path')

      consola.track('UPLOAD_FILE')

      this.addFiles(paths)

      event.target.value = ''
    },

    /**
     *
     */
    openUrl() {
      if (isEmpty(this.webAddress) || (!startsWith(this.webAddress, 'http://') && !startsWith(this.webAddress, 'https://'))) {
        throw new Warning('Upload failed.', 'Please enter a valid web address.')
      }

      Nudify.addUrl(this.webAddress)

      consola.track('UPLOAD_URL')

      this.webAddress = ''
    },

    /**
     *
     */
    async openInstagramPhoto() {
      if (isEmpty(this.instagramPhoto)) {
        throw new Warning('Upload failed.', 'Please enter a valid Instagram photo.')
      }

      let post

      try {
        post = await instagram.getPost(this.instagramPhoto)
      } catch (error) {
        throw new Warning(
          'Upload failed.',
          'Unable to download the photo, please verify that the address is correct and that you are connected to the Internet.',
          error,
        )
      }

      Nudify.addUrl(post.downloadUrl)

      consola.track('UPLOAD_INSTAGRAM')

      this.instagramPhoto = ''
    },
  },
}
</script>

<style lang="scss" scoped>
.uploader {
}

.uploader__methods {
  @apply flex-1;
}

.method__header {
  @apply mb-9;

  .title {
    @apply text-lg font-semibold text-generic-100;

    .icon {
      @apply mr-2;
    }
  }

  .subtitle {
    @apply font-light mb-6;

    .help {
      @apply ml-2;
      cursor: help;
    }
  }
}

.method__body {
  &.method__body--columns {
    @apply flex flex-wrap;

    .column {
      @apply w-1/2 px-2;

      .box {
        @apply h-full;
      }
    }
  }

  .input {
    @apply mb-3;
  }

}
</style>
