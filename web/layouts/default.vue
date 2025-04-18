<template>
  <v-app>
    <v-divider v-if="$store.getters.isLocal" class="py-1 warning"></v-divider>
    <v-navigation-drawer
      v-if="$vuetify.breakpoint.lgAndUp && hasDrawer"
      :width="400"
      app
      fixed
    >
      <template #prepend>
        <v-divider
          v-if="$store.getters.isLocal"
          class="py-1 warning"
        ></v-divider>
        <message-thread-header></message-thread-header>
        <div class="overflow-y-auto v-navigation-drawer__message-thread">
          <message-thread></message-thread>
        </div>
      </template>
    </v-navigation-drawer>
    <v-main :class="{ 'has-drawer': hasDrawer && $vuetify.breakpoint.lgAndUp }">
      <toast></toast>
      <Nuxt v-if="$store.getters.authStateChanged" />
      <loading-dashboard v-else></loading-dashboard>
    </v-main>
  </v-app>
</template>

<script lang="ts">
import { Vue, Component } from 'vue-property-decorator'
import { mdiBullhorn } from '@mdi/js'
import { setAuthHeader } from '~/plugins/axios'

@Component
export default class DefaultLayout extends Vue {
  poller: number | null = null
  mdiBullhorn: string = mdiBullhorn

  get hasDrawer(): boolean {
    return ['threads', 'threads-id'].includes(this.$route.name ?? '')
  }

  mounted() {
    // this.startPoller()
    setTimeout(
      () => {
        if (this.poller) {
          clearInterval(this.poller)
        }
      },
      60 * 1000 * 60,
    )
  }

  beforeDestroy(): void {
    if (this.poller) {
      clearInterval(this.poller)
    }
  }

  startPoller() {
    this.poller = window.setInterval(async () => {
      await this.$store.dispatch('setPolling', true)

      const promises = []
      if (this.$store.getters.getAuthUser && this.$store.getters.getOwner) {
        setAuthHeader((await this.$fire.auth.currentUser?.getIdToken()) ?? '')
        promises.push(
          this.$store.dispatch('loadThreads'),
          this.$store.dispatch('getHeartbeat'),
        )
      }

      if (this.$store.getters.getAuthUser) {
        promises.push(this.$store.dispatch('loadPhones', true))
      }

      if (this.$store.getters.hasThread && this.$store.getters.getUser) {
        promises.push(
          this.$store.dispatch(
            'loadThreadMessages',
            this.$store.getters.getThread.id,
          ),
        )
      }
      await Promise.all(promises)

      setTimeout(() => {
        this.$store.dispatch('setPolling', false)
      }, 1000)
    }, 10000)
  }
}
</script>

<style lang="scss">
.v-application {
  .w-full {
    width: 100%;
  }
  .h-full {
    height: 100%;
  }

  .has-drawer {
    .v-snack {
      padding-left: 400px;
    }
  }

  .v-navigation-drawer__message-thread {
    height: calc(100vh - 120px);

    /* width */
    &::-webkit-scrollbar {
      width: 8px;
    }

    /* Track */
    &::-webkit-scrollbar-track {
      background: #363636;
    }

    /* Handle */
    &::-webkit-scrollbar-thumb {
      background: #666666;
      border-radius: 8px;
    }
  }
  code.hljs {
    font-size: 16px;
  }
}

.feedback-btn {
  position: fixed;
  z-index: 15;
  right: -56px;
  margin: 0;
  top: 45%;
  border-bottom-left-radius: 0;
  border-bottom-right-radius: 0;
  transform: rotate(-90deg);
  -moz-transform: rotate(-90deg);
  -ms-transform: rotate(-90deg);
  -o-transform: rotate(-90deg);
  -webkit-transform: rotate(-90deg);
}
</style>
