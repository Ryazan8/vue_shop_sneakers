<script setup>
import Card from "./Card.vue";
import {inject} from "vue";

defineProps({
  items: Array
})

const onClickAdd = () => {
  alert('111')
}

const addToFavorite = inject('addToFavorite')

</script>

<template>

  <div class="grid grid-cols-4 gap-5 ">

    <Card
        v-for="item in items"
        :id="item.id"
        :key="item.id"
        :title="item.title"
        :imageUrl="item.imageUrl"
        :price="item.price"
        :onClickAdd="onClickAdd"
        :onClickFavorite="() => addToFavorite(item)"
        :isFavorite="item.isFavorite"
    />

  </div>

</template>
