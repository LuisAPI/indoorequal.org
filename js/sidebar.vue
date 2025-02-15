<template>
  <v-navigation-drawer
    :value="value !== null"
    width="350"
    temporary
    app
    touchless
    @input="updateValue"
  >
    <component
      v-if="hasSubSidebar"
      :is="componentName"
      @openPreview="(file) => this.$emit('openPreview', file)"
      @hideMenu="hideMenu"
      @close="updateMenu('')"
    />
    <template v-else>
      <div class="pa-3">
        <div class="d-flex align-center justify-space-between">
          <h1 class="display-1">
            <img
              :src="logo"
              alt="indoor="
              class="logo"
            />
          </h1>
          <v-tooltip bottom>
            <template v-slot:activator="{ on }">
              <v-btn
                icon
                @click="hideMenu"
                v-on="on"
              >
                <v-icon>{{ mdiArrowExpandLeft }}</v-icon>
              </v-btn>
            </template>
            <span>{{ $t("sidebar.collapse") }}</span>
          </v-tooltip>
        </div>
        <i18n path="sidebar.description" tag="p" class="mt-2 mb-1">
          <a href="https://openstreetmap.org/" target="_blank">{{ $t('sidebar.osm') }}</a>
        />
        </i18n>
      </div>
      <v-list>
        <v-list-item @click="display('explore')">
          <v-list-item-icon>
            <v-icon>{{ mdiMapMarkerCircle }}</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>{{ $t('sidebar.explore.title') }}</v-list-item-title>
            <v-list-item-subtitle>{{ $t('sidebar.explore.subtitle') }}</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click="display('api')">
          <v-list-item-icon>
            <v-icon>{{ mdiMap }}</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>{{ $t('sidebar.api.title') }}</v-list-item-title>
            <v-list-item-subtitle>{{ $t('sidebar.api.subtitle') }}</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click="display('news')">
          <v-list-item-icon>
            <v-badge
              :value="hasNews"
              dot
            >
              <v-icon>{{ mdiNewspaperVariantOutline }}</v-icon>
            </v-badge>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>{{ $t('sidebar.news.title') }}</v-list-item-title>
            <v-list-item-subtitle>{{ $t('sidebar.news.subtitle') }}</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
        <v-list-item @click="display('about')">
          <v-list-item-icon>
            <v-icon>{{ mdiInformationOutline }}</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>{{ $t('sidebar.about.title') }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>
        <v-divider></v-divider>
        <v-list-item @click="display('preview')">
          <v-list-item-icon>
            <v-icon>{{ mdiPuzzleCheck }}</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>
              <v-chip
                color="primary"
                x-small
              >New</v-chip>
              {{ $t('sidebar.preview.title') }}
            </v-list-item-title>
            <v-list-item-subtitle>{{ $t('sidebar.preview.subtitle') }}</v-list-item-subtitle>
          </v-list-item-content>
        </v-list-item>
        <v-list-item
          href="https://taginfo.indoorequal.org"
          target="_blank"
        >
          <v-list-item-icon>
            <v-icon>{{ mdiTag }}</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>{{ $t('sidebar.taginfo.title') }}</v-list-item-title>
            <v-list-item-subtitle>{{ $t('sidebar.taginfo.subtitle') }}</v-list-item-subtitle>
          </v-list-item-content>
          <v-list-item-icon><v-icon>{{ mdiOpenInNew }}</v-icon></v-list-item-icon>
        </v-list-item>
        <v-list-item
          href="https://wiki.openstreetmap.org/wiki/Simple_Indoor_Tagging"
          rel="noopener"
          target="_blank"
        >
          <v-list-item-icon>
            <v-icon>{{ mdiPuzzleEditOutline }}</v-icon>
          </v-list-item-icon>
          <v-list-item-content>
            <v-list-item-title>{{ $t('sidebar.learn.title') }}</v-list-item-title>
            <v-list-item-subtitle>{{ $t('sidebar.learn.subtitle') }}</v-list-item-subtitle>
          </v-list-item-content>
          <v-list-item-icon><v-icon>{{ mdiOpenInNew }}</v-icon></v-list-item-icon>
        </v-list-item>
      </v-list>
      <i18n
        v-if="replicationStatus"
        path="sidebar.last_update"
        class="pa-3 caption"
        tag="div"
      >
        <time
          :datetime="lastUpdateTimestamp"
          :title="lastUpdateTimestamp"
        >{{ lastUpdateTimestampFormatted }}</time>
        <a
          href="https://status.indoorequal.org/"
          target="_blank"
          rel="noopener"
        >{{ $t('sidebar.status') }}</a>
      </i18n>
    </template>
  </v-navigation-drawer>
</template>

<script>
import {
  mdiArrowExpandLeft,
  mdiInformationOutline,
  mdiMap,
  mdiMapMarkerCircle,
  mdiNewspaperVariantOutline,
  mdiOpenInNew,
  mdiPuzzleCheck,
  mdiPuzzleEditOutline,
  mdiTag,
} from '@mdi/js';
import dayjs from 'dayjs';
import relativeTime from 'dayjs/plugin/relativeTime';
import { tilesUrl } from '../config.json';
import news from './news';
import logo from 'data-url:../icons/indoorequal.svg';

dayjs.extend(relativeTime);

const COMPONENTS = {
  explore: () => import('./explore_list'),
  api: () => import('./api_info'),
  about: () => import('./about_info'),
  news: () => import('./news_info'),
  preview: () => import('./preview/preview_sidebar'),
};

const fetchReplicationStatus = async function() {
  const replicationStatusRequest = await fetch(`${tilesUrl}replicationstatus`);
  return replicationStatusRequest.json();
}

export default {
  mixins: [news],

  props: {
    value: {
      type: String,
    }
  },

  data() {
    return {
      mdiArrowExpandLeft,
      mdiInformationOutline,
      mdiMap,
      mdiMapMarkerCircle,
      mdiNewspaperVariantOutline,
      mdiOpenInNew,
      mdiPuzzleCheck,
      mdiPuzzleEditOutline,
      mdiTag,
      logo,
      replicationStatus: null
    };
  },

  computed: {
    hasSubSidebar() {
      return this.value !== '';
    },
    componentName() {
      return COMPONENTS[this.value];
    },
    lastUpdateTimestamp() {
      return this.replicationStatus.timestamp;
    },
    lastUpdateTimestampFormatted() {
      return dayjs(this.lastUpdateTimestamp).fromNow();
    }
  },

  methods: {
    display(component) {
      this.updateMenu(component)
      plausible(`Display ${component}`);
    },

    hideMenu() {
      this.updateMenu(null);
    },

    updateValue(value) {
      if (!value) {
        this.updateMenu(null);
      }
    },

    updateMenu(value) {
      this.$emit('input', value);
    }
  },

  watch: {
    value: {
      immediate: true,
      async handler(value) {
        if (value !== null && value !== '' && !Object.keys(COMPONENTS).includes(value)) {
          this.updateMenu('');
        }
        if (value !== null && !this.replicationStatus) {
          this.replicationStatus = await fetchReplicationStatus();
        }
      }
    }
  }
}
</script>

<style scoped>
.logo {
  max-width: 70%;
}
</style>
