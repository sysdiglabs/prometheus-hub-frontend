<template>
  <b-row>
    <b-col>
      <b-img
        class="logo-image"
        :src="appIcon"
        :alt="appName"
      />
      <div class="versions">
        <h2 class="version-title">
          Supported Versions
        </h2>
        <b-dropdown
          id="version-selector"
          :text="`Version ${currentVersion}`"
          class="version-selector m-md-2"
          :disabled="appVersions.length <= 1"
          variant="outline-primary"
        >
          <b-dropdown-item
            v-for="version in versionsWithoutCurrent"
            :key="version"
            :class="{current: version === currentVersion}"
            :to="`/apps/${appId}/${version}`"
            class="version"
          >
            Version {{ version }}
          </b-dropdown-item>
        </b-dropdown>
      </div>
      <b-tabs
        class="tabs"
        v-model="tabIndex"
        fill
        justified
        content-class="tabs-content mt-3"
      >
        <b-tab
          v-for="resource in appResources"
          :key="resource.kind"
          :title="tabTitle(resource.kind)"
          class="tab"
          lazy
        >
          <AppPresentation class="app-presentation" :resource="resource" :resources="appResources" />
        </b-tab>
      </b-tabs>
    </b-col>
  </b-row>
</template>

<script>
import { mapState } from 'vuex'
import AppPresentation from '@/components/AppPresentation'
import { getCanonicalForApp } from '@/infrastructure/Canonical'

export default {
  data () {
    return {
      tabIndex: 0
    }
  },
  components: {
    AppPresentation
  },
  layout: 'content',
  async fetch ({ store, params }) {
    await store.dispatch('getAppAndResourcesByVersion', { id: params.id, version: params.version })
  },
  created () {
    if (!this.$root._route.hash) {
      return
    }

    const visibleTab = this.$root._route.hash.replace('#', '')
    const findTabIndex = (resources, candidate) => {
      for (let index = 0; index < resources.length; index++) {
        if (resources[index].kind === candidate) {
          return index
        }
      }
      return 0
    }

    this.tabIndex = findTabIndex(this.appResources, visibleTab)
  },
  computed: {
    ...mapState({
      appResources: state => state.appResources,
      app: state => state.app
    }),
    appId () {
      return this.app.id
    },
    appName () {
      return this.app.name
    },
    appVersions () {
      return this.app.availableVersions
    },
    appIcon () {
      return this.app.icon
    },
    currentVersion () {
      return this.$route.params.version || this.appVersions[this.appVersions.length - 1]
    },
    versionsWithoutCurrent () {
      return this.appVersions.filter(version => version !== this.currentVersion)
    }
  },
  methods: {
    copy (text) {
      navigator.clipboard.writeText(text)
    },
    tabTitle (kind) {
      switch (kind) {
        case 'RecordingRule':
          return 'Recording Rules'
        case 'Dashboard':
          return 'Dashboards'
        case 'Alert':
          return 'Alerts'
        default:
          return kind.split(/(?=[A-Z])/).join(' ')
      }
    }
  },
  head () {
    return {
      title: `Monitoring ${this.appName} with Prometheus: Configuration, dashboards and alerts.`,
      meta: [
        { hid: 'description', name: 'description', content: `${this.appName} app details` }
      ],
      link: [
        {
          rel: 'canonical',
          href: getCanonicalForApp(this.appId, this.appVersions, this.currentVersion)
        }
      ]
    }
  }
}
</script>

<style scoped lang="scss">
.title {
  font-size: 36px;
  line-height: 36px;
  margin-bottom: 40px;
}
.logo-image{
  height: 70px;
}
.versions {
  margin-bottom: 32px;
  ::v-deep .version-selector {
    margin: 0 !important;

    .btn {
      background-color: transparent;
      border: 1px solid $secondary;
      color: $primary;
      width: 320px;
      height: 48px;
      text-align: left;
      &:not(.disabled):hover {
        border: 1px solid $primary;
      }
      &.disabled:hover {
        cursor: default;
      }
      &:after {
        float: right;
        color: $primary;
        margin-top: 10px;
      }
      &.disabled:after {
        display: none;
      }
    }
    .dropdown-item {
      color: $primary;
      padding: 0.375rem 0.75rem;
      &:hover {
        background-color: $primary;
        color: white;
      }
    }
    .dropdown-menu {
      border: 1px solid $primary;
      width: 320px;
      height: 48px;
    }
  }
  .version-title {
    text-transform: uppercase;
    font-size: $font-L;
    line-height: 19px;
    letter-spacing: 1px;
    &:after {
      content: " ";
      border-top: 2px solid $primary;
      margin-top: $separator-margin;
      margin-bottom: $separator-margin;
      display: block;
      max-width: $separator-width;
    }
  }
}
.tabs {
  ::v-deep .nav-tabs {
    border-bottom: 1px solid $secondary;
    .nav-link {
      text-transform: uppercase;
      font-size: $font-L;
      line-height: 19px;
      letter-spacing: 1px;
      color: $secondary;
      text-align: center;
      padding-bottom: 16px;
      &:hover {
        border: none;
        color: $primary;
      }
      &:focus {
        border-color: $primary;
      }
      &.active {
        color: black;
        border-color: $primary;
        border-style: solid;
        border-width: 0 0 8px 0;
        padding-bottom: 16px;
      }
    }
  }
}
.app-presentation {
  ::v-deep {
    .markdown {
      font-size: 18px;
      line-height: 24px;
    }
      .maintainers {
      .maintainers-title {
        display: inline-block;
        margin-right: 1rem;
      }
      .maintainer {
        font-weight: $weight-bold;
      }
    }
  }
}

</style>
