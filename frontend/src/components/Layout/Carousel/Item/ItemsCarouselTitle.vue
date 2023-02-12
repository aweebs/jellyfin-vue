<template>
  <div>
    <router-link v-if="logo && logo.tag && logoLink" :to="logoLink">
      <v-img
        class="mb-2"
        :max-width="$vuetify.display.mdAndUp ? '50%' : '40%'"
        :max-height="$vuetify.display.smAndUp ? '7.5em' : '4em'"
        contain
        data-swiper-parallax="-300"
        :src="logo.url" />
    </router-link>
    <router-link
      v-else-if="itemLink && titleString"
      data-swiper-parallax="-300"
      class="link d-block text-h4 text-sm-h3 text-sm-h2 text-truncate"
      :to="itemLink">
      {{ titleString }}
    </router-link>
    <p
      v-if="subtitle"
      data-swiper-parallax="-200"
      class="text-truncate text-subtitle-2">
      {{ subtitle }}
    </p>
    <h2
      v-if="item.Taglines && item.Taglines.length > 0"
      data-swiper-parallax="-200"
      class="text-truncate">
      {{ item.Taglines[0] }}
    </h2>
  </div>
</template>

<script setup lang="ts">
import { computed, ref, watch } from 'vue';
import { useI18n } from 'vue-i18n';
import { BaseItemDto } from '@jellyfin/sdk/lib/generated-client';
import { getLogo, ImageUrlInfo } from '@/utils/images';
import { getItemDetailsLink } from '@/utils/items';

const props = defineProps<{ item: BaseItemDto }>();

const { t } = useI18n();

const itemLink = ref('');
const titleString = ref('');
const logoLink = ref('');
const subtitle = ref('');

const logo = computed(() => getLogo(props.item));

watch(
  () => props.item,
  (item) => {
    switch (item.Type) {
      case 'MusicAlbum': {
        if (item.AlbumArtists?.length) {
          logoLink.value = getItemDetailsLink(
            item.AlbumArtists[0],
            'MusicArtist'
          );
        }

        if (item.AlbumArtist) {
          titleString.value = item.AlbumArtist;
        }

        if (item.Name) {
          subtitle.value = item.Name;
        }

        break;
      }
      case 'Episode': {
        if (item.SeriesId) {
          logoLink.value = getItemDetailsLink({ Id: item.SeriesId }, 'Series');
        }

        if (item.SeasonName && item.IndexNumber && item.Name) {
          const episodeString = t('episodeNumber', {
            episodeNumber: item.IndexNumber
          });

          subtitle.value = `${item.SeasonName} - ${episodeString}\n${item.Name}`;
        }

        if (item.SeriesName) {
          titleString.value = item.SeriesName;
        }

        break;
      }
    }

    /**
     * Instead of using 'default', we need this additional extra check
     * in case an Album doesn't have artists, for example.
     */
    if (itemLink.value === '') {
      itemLink.value = getItemDetailsLink(item);
    }

    if (titleString.value === '' && item.Name) {
      titleString.value = item.Name;
    }
  },
  { immediate: true }
);
</script>
