<script setup>
import {onMounted, reactive, provide, ref, watch} from "vue";
import axios from "axios";


import Header from '@/components/Header.vue'
import CardList from '@/components/CardList.vue'
import Drawer from "@/components/Drawer.vue";

const items = ref([])

const drawerOpen = ref(false)
const closeDrawer = () => {
  drawerOpen.value = false
}
const openDrawer = () => {
  drawerOpen.value = true
}

const filters = reactive({
  sortBy: 'title',
  searchQuery: '',
})

const onChangeSelect = (e) => {
  filters.sortBy = e.target.value
}
const onChangeSearchInput = (e) => {
  filters.searchQuery = e.target.value
}

const fetchFavorites = async () => {
  try {

    const { data: favorites } = await axios.get('https://55062b80b076a7f3.mokky.dev/favorites')
    items.value = items.value.map(item => {
      const favorite = favorites.find(favorite => favorite.parentId === item.id)

      if (!favorite) {
        return item
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (err) {
    console.log(err);
  }
}

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        parentId: item.id
      }

      item.isFavorite = true
      const { data } = await axios.post('https://55062b80b076a7f3.mokky.dev/favorites', obj)
      item.favoriteId = data.id
    } else {
      item.isFavorite = false
      await axios.delete(`https://55062b80b076a7f3.mokky.dev/favorites/${item.favoriteId}`)
      item.favoriteId = null
    }
  } catch (err) {
    console.log(err);
  }
}

const fetchItems = async (item) => {
  try {
    const params = {
      sortBy: filters.sortBy,
    }

    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }

    const {data} = await axios.get('https://55062b80b076a7f3.mokky.dev/items',
        {
          params
        }
    )
    items.value = data.map(obj => ({
      ...obj,
      isFavorites: false,
      favoriteId: null,
      isAdded: false
    }))
  } catch (err) {
    console.log(err);
  }
}

onMounted(async () => {
  await fetchItems()
  await fetchFavorites()
})
watch(filters, fetchItems)

provide('cartActions', {
  closeDrawer,
  openDrawer
})
</script>

<template>
    <Drawer v-if="drawerOpen" />

  <div class="bg-white rounded-xl shadow-xl mt-14 w-4/5 m-auto">

    <Header @open-drawer="openDrawer" />

    <div class="p-10">
      <div class="flex justify-between items-center">
        <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

        <div class="flex gap-4">
          <select @change="onChangeSelect" class="py-2 px-3 border rounded-md outline-none">
            <option value="name">По названию</option>
            <option value="price">По цене (дешёвые)</option>
            <option value="-price">По цене (дорогие)</option>
          </select>

          <div class="relative">
            <img class="absolute left-4 top-3" src="/search.svg" alt="Search">
            <input
                @input="onChangeSearchInput"
                type="text"
                placeholder="Поиск..."
                class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
            >
          </div>
        </div>
      </div>

      <div class="mt-10">
        <CardList :items="items" @add-to-favorite="addToFavorite" />
      </div>

    </div>


  </div>
</template>


<style scoped>

</style>
