<script setup lang='ts'>
import copyText from 'copy-text-to-clipboard'
import { useRoute, useRouter } from 'vue-router'
import { activeMode, bags, getSearchResults, iconSize, isCurrentCollectionLoading, listType, showHelp, toggleBag } from '../store'
import { isLocalMode } from '../env'
import { cacheCollection } from '../data'
import { getIconSnippet } from '../utils/icons'

const { search, icons, category, collection } = getSearchResults()
const showBag = ref(false)
const copied = ref(false)

const maxMap = new Map<string, number>()
const current = ref('')
const max = ref(isLocalMode ? 500 : 200)

const onCopy = (status: boolean) => {
  copied.value = status
  setTimeout(() => {
    copied.value = false
  }, 2000)
}

const toggleCategory = (cat: string) => {
  if (category.value === cat)
    category.value = ''
  else category.value = cat
}

const namespace = computed(() =>
  !collection.value || collection.value.id === 'all'
    ? ''
    : `${collection.value.id}:`,
)

const url = computed(() => collection.value?.url || collection.value?.author?.url)

const onSelect = async(icon: string) => {
  switch (activeMode.value) {
    case 'select':
      toggleBag(icon)
      break
    case 'copy':
      onCopy(copyText(await getIconSnippet(icon, 'id', true) || icon))
      break
    default:
      current.value = icon
      break
  }
}

watch(
  namespace,
  () => {
    max.value = maxMap.get(namespace.value) || 200
  },
)

const loadMore = () => {
  max.value += 100
  maxMap.set(namespace.value, max.value)
}

const loadAll = async() => {
  if (!namespace.value)
    return

  await cacheCollection(collection.value!.id)
  max.value = icons.value.length
  maxMap.set(namespace.value, max.value)
}

const loading = isCurrentCollectionLoading()

const route = useRoute()
const router = useRouter()
onMounted(() => {
  search.value = route.query.s as string || ''
  watch([search, collection], () => {
    router.replace({ query: { s: search.value } })
  })
})
</script>

<template>
  <WithNavbar>
    <div class="flex flex-auto h-full overflow-hidden">
      <Drawer class="h-full overflow-y-overlay flex-none hidden md:block" style="width:220px" />
      <div v-if="collection" class="py-5 h-full overflow-y-overlay flex-auto overflow-x-hidden relative">
        <!-- Loading -->
        <div
          class="absolute top-0 left-0 right-0 bottom-0 bg bg-opacity-75 content-center transition-opacity duration-100 z-50"
          :class="loading ? '' : 'opacity-0 pointer-events-none'"
        >
          <div class="absolute" style="top:50%;left:50%;transform:translate(-50%,-50%)">
            Loading...
          </div>
        </div>

        <div class="flex px-20px">
          <!-- Left -->
          <div class="flex-auto px-2">
            <NavPlaceholder class="md:hidden" />

            <div class="text-black text-20px flex select-none">
              <div class="whitespace-no-wrap overflow-hidden">
                {{ collection.name }}
              </div>
              <!-- <a
                v-if="url"
                class="ml-1 mt-1 text-base opacity-25 hover:opacity-100"
                :href="url"
                target="_blank"
              >
                <Icon icon="la:external-link-square-alt-solid" />
              </a> -->
              <div class="flex-auto" />
            </div>
            <div class="text-12px block opacity-65">
              {{ collection.author?.name }}
            </div>
            <div v-if="collection.license">
              <a
                class="text-12px opacity-65 hover:opacity-100"
                :href="collection.license.url"
                target="_blank"
              >{{ collection.license.title }}</a>
            </div>
          </div>

          <!-- Right -->
          <div class="flex flex-col">
            <ActionsMenu :collection="collection" />
            <div class="flex-auto" />
          </div>
        </div>

        <!-- Categories -->
        <div class="py-2 px-7 overflow-x-overlay flex flex-no-wrap select-none">
          <template v-if="collection.categories">
            <div
              v-for="c of Object.keys(collection.categories)"
              :key="c"
              class="
                whitespace-nowrap text-sm inline-block px-2 border border-gray-200 rounded-full m-1 hover:bg-gray-50 cursor-pointer
                dark:border-dark-200 dark:hover:bg-dark-200
              "
              :class="c === category ? 'text-primary border-primary dark:border-primary' : 'opacity-75'"
              @click="toggleCategory(c)"
            >
              {{ c }}
            </div>
          </template>
        </div>

        <!-- Searching -->
        <div class="mx-8 my-2 hidden md:flex shadow rounded-6px outline-none py-1 px-12px bg-white">
          <Icon icon="ic:baseline-search" class="m-auto flex-none text-20px" />
          <form action="/collection/all" class="flex-auto" role="search" method="get" @submit.prevent>
            <input
              v-model="search"
              aria-label="Search"
              class="text-16px font-400 outline-none w-full py-1 px-3 m-0 bg-transparent"
              name="s"
              placeholder="Search..."
            >
          </form>

          <Icon v-if="search" icon="carbon:close" class="m-auto text-lg -mr-1 opacity-60" @click="search = ''" />
        </div>

        <!-- Icons -->
        <div class="px-4 pt-2 pb-4 text-center">
          <Icons
            :icons="icons.slice(0, max)"
            :selected="bags"
            class="text-36px"
            :display="listType"
            :search="search"
            :namespace="namespace"
            @select="onSelect"
          />
          <button v-if="icons.length > max" class="btn mx-1 my-3" @click="loadMore">
            Load More
          </button>
          <button v-if="icons.length > max && namespace" class="btn mx-1 my-3" @click="loadAll">
            Load All ({{ icons.length - max }})
          </button>
          <p class="color-fade text-sm pt-4">
            {{ icons.length }} icons
          </p>
        </div>

        <Footer />

        <!-- Bag Fab -->
        <FAB
          v-if="bags.length"
          icon="carbon:shopping-bag"
          :number="bags.length"
          @click="showBag = true"
        />

        <!-- Bag -->
        <Modal :value="showBag" direction="right" @close="showBag = false">
          <Bag
            @close="showBag = false"
            @select="onSelect"
          />
        </Modal>

        <!-- Details -->
        <Modal :value="!!current" @close="current = ''">
          <IconDetail :icon="current" :show-collection="collection.id === 'all'" @close="current = ''" @copy="onCopy" />
        </Modal>

        <!-- Help -->
        <ModalDialog :value="showHelp" @close="showHelp = false">
          <HelpPage />
        </ModalDialog>

        <!-- Mode -->
        <div
          class="fixed top-0 right-0 pl-4 pr-2 py-1 rounded-l-full bg-primary text-white shadow mt-16 cursor-pointer transition-transform duration-300 ease-in-out"
          :style="activeMode !== 'normal' ? {} : { transform: 'translateX(120%)' }"
          @click="activeMode = 'normal'"
        >
          {{ activeMode === 'select' ? 'Multiple select' : 'Name copying mode' }}
          <Icon icon="carbon:close" class="inline-block text-xl align-text-bottom" />
        </div>

        <SearchElectron />

        <Notification :value="copied">
          <!-- <Icon icon="mdi:check" class="inline-block mr-16px font-xl align-middle" /> -->
          <img src="@/fonts/check.svg" class="inline-block mr-16px">
          <span class="align-middle">Copied to clipboard!</span>
        </Notification>
      </div>
    </div>
  </WithNavbar>
</template>
