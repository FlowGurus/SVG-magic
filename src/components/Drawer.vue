<script setup lang='ts'>
import { sortedCollectionsInfo } from '../data'
import { isFavorited, toggleFavorite } from '../store'
import { isElectron } from '../env'

const route = useRoute()
const current = computed(() => route.path.split('/').slice(-1)[0])

const collections = computed(() => {
  return [
    { id: 'all', name: 'All' },
    ...sortedCollectionsInfo.value,
  ]
})
</script>

<template>
  <div class="bg-grey">
    <NavPlaceholder class="mb-4" />
    <div
      v-if="!isElectron"
      sticky top-0 z-1
    >
      <button
        v-show="$route.path !== '/'"
        icon-button text-xl px-4 py-4
        @click="$router.replace('/')"
      >
        <div i-carbon:arrow-left />
      </button>
    </div>

    <!-- Collections -->
    <RouterLink
      v-for="collection in collections"
      :key="collection.id"
      class="px-4 py-2 flex"
      :to="`/collection/${collection.id}`"
      active-class="bg"
    >
      <div
        class="flex-auto py-1 "
        :class="collection.id === current ? 'text-primary' : ''"
      >
        <div class="text-base leading-tight font-500">
          {{ collection.name }}
        </div>
        <div class="text-xs block opacity-50 mt-1">
          {{ collection.id !== 'all' ? `${collection.total} icons` : `${collections.length} iconsets` }}
        </div>
      </div>
      <button
        v-if="collection.id !== 'all'"
        icon-button
        :class="isFavorited(collection.id) ? 'op50 hover:op100' : 'op0 hover:op50' "
        class="flex-none text-lg p0.5 -mr-1 hover:text-primary flex"
        @click="toggleFavorite(collection.id)"
      >
        <div :class="isFavorited(collection.id) ? 'i-carbon-star-filled' : 'i-carbon-star'" ma />
      </button>
    </RouterLink>
  </div>
</template>
