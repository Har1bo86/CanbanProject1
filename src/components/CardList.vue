<template>
  <section :style="`background: ${options.color}`" @drop="onDrop($event, options.id)" @dragover.prevent
    @dragenter.prevent>

    <div class="sorting-panel">
      <v-btn icon="mdi-sort-ascending" @click="sortAscending" title="Сортировать по возрастанию рейтинга" />
      <v-btn icon="mdi-sort-descending" @click="sortDescending" title="Сортировать по убыванию рейтинга" />
      <v-btn icon="mdi-sort" @click="resetSorting" title="Сбросить сортировку" />
    </div>

    <div class="title">
      <h2>
        {{ options.title }}
      </h2>
      <div class="counter">
        <span>{{ cards.length }}</span>
      </div>
    </div>

    <v-btn icon="mdi-plus" variant="tonal" class="mt-5" color="white" @click="isNewCardDialogOpen = true" />

    <CardItem v-for="(card, index) in cards" draggable="true" :key="index" :card="card" :options="props.options"
      @delete-card="deleteCard(card.id)" @dragstart="onDragStart($event, card)" />

    <CardForm title="Добавление новой карточки" v-model="isNewCardDialogOpen" :form="form" @save-card="addCard"
      @close-form="isNewCardDialogOpen = false" />
  </section>
</template>

<script setup>
import { ref, inject, watch } from 'vue';
import CardItem from './CardItem.vue';
import CardForm from './CardForm.vue';

const firstList = inject('firstList');
const secondList = inject('secondList');
const lastList = inject('lastList');

const props = defineProps({
  options: {},
});

const isNewCardDialogOpen = ref(false);

const form = ref({
  image: '',
  id: null,
  title: '',
  description: '',
  category: '',
  price: null,
  rating: {},
});

let cards = ref([]);

function getLocalCards() {
  switch (props.options.id) {
    case 1:
      cards = firstList;
      break;
    case 2:
      cards = secondList;
      break;
    case 3:
      cards = lastList;
      break;
  }
  sortById(); // сортируем по id после получения карточек
}
getLocalCards();

watch(() => props.options.id, getLocalCards);

function addCard() {
  cards.value.unshift(form.value);
  isNewCardDialogOpen.value = false;
  sortById(); // сортируем по id после добавления новой карточки
}

function deleteCard(id) {
  cards.value = cards.value.filter((card) => card.id != id);
}

function onDragStart(event, item) {
  event.dataTransfer.dropEffect = 'move';
  event.dataTransfer.effectAllowed = 'move';
  event.dataTransfer.setData('itemId', item.id.toString());
}

function onDrop(event, optionsId) {
  const itemId = parseInt(event.dataTransfer.getData('itemId'));
  let otherLists = [];

  switch (optionsId) {
    case 1:
      cards.value = firstList.value;
      otherLists = secondList.value.concat(lastList.value);
      break;
    case 2:
      cards.value = secondList.value;
      otherLists = firstList.value.concat(lastList.value);
      break;
    case 3:
      cards.value = lastList.value;
      otherLists = secondList.value.concat(firstList.value);
      break;
  }

  const newCard = otherLists.find((card) => card.id === itemId);

  firstList.value = firstList.value.filter((card) => card.id !== newCard.id);
  secondList.value = secondList.value.filter((card) => card.id !== newCard.id);
  lastList.value = lastList.value.filter((card) => card.id !== newCard.id);
  cards.value.unshift(newCard);

  switch (optionsId) {
    case 1:
      firstList.value = cards.value;
      break;
    case 2:
      secondList.value = cards.value;
      break;
    case 3:
      lastList.value = cards.value;
      break;
  }
  sortById(); // сортируем по id после перетаскивания карточек
}

function sortAscending() {
  cards.value.sort((a, b) => a.rating.rate - b.rating.rate);
}

function sortDescending() {
  cards.value.sort((a, b) => b.rating.rate - a.rating.rate);
}

function sortById() {
  cards.value.sort((a, b) => a.id - b.id);
}

function resetSorting() {
  sortById(); // сортируем по id при сбросе
}
</script>

<style lang="scss" scoped>
section {
  padding: 10px;
  width: 400px;
  min-height: 500px;
  height: fit-content;
  border-radius: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;

  .sorting-panel {
    display: flex;
    justify-content: center;
    gap: 10px;
    margin-bottom: 10px;
  }

  .title {
    padding-bottom: 10px;
    width: 90%;
    display: flex;
    align-items: center;
    justify-content: center;
    border-bottom: 2px solid white;

    .counter {
      margin-left: 10px;
      background: white;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      display: flex;
      justify-content: center;
      align-items: center;
    }
  }
}
</style>
