<template>
  <section :style="`background: ${options.color}`" @drop="onDrop($event, options.id)" @dragover.prevent
    @dragenter.prevent>
    <!-- Панель сортировки -->
    <div class="sorting-panel">
      <v-btn icon="mdi-sort-ascending" @click="sortCards('asc')" color="blue" class="sort-btn">
        <span class="btn-text">По возрастанию рейтинга</span>
        ▲
      </v-btn>
      <v-btn icon="mdi-sort-descending" @click="sortCards('desc')" color="blue" class="sort-btn">
        <span class="btn-text">По убыванию рейтинга</span>
        ▼
      </v-btn>
      <v-btn icon="mdi-refresh" @click="sortCards('default')" color="blue" class="sort-btn">
        <span class="btn-text">Без сортировки</span>
        ⏏
      </v-btn>
    </div>

    <!-- Заголовок и счетчик -->
    <div class="title">
      <h2>
        {{ options.title }}
      </h2>
      <div class="counter">
        <span>{{ cards.length }}</span>
      </div>
    </div>

    <!-- Кнопка добавления карточки -->
    <v-btn icon="mdi-plus" variant="tonal" class="mt-5" color="white" @click="isNewCardDialogOpen = true" />

    <!-- Список карточек -->
    <CardItem v-for="(card, index) in cards" draggable="true" :key="index" :card="card" :options="props.options"
      @delete-card="deleteCard(card.id)" @dragstart="onDragStart($event, card)" />

    <!-- Форма добавления новой карточки -->
    <CardForm title="Добавление новой карточки" v-model="isNewCardDialogOpen" :form="form" @save-card="addCard"
      @close-form="isNewCardDialogOpen = false" />
  </section>
</template>

<script setup>
import { ref, inject } from 'vue';
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
}
getLocalCards();

function addCard() {
  cards.value.unshift(form.value);
  isNewCardDialogOpen.value = false;
}

function deleteCard(id) {
  cards.value = cards.value.filter((card) => card.id != id);
}

function sortCards(order) {
  getLocalCards();
  if (order === 'asc') {
    cards.value = [...cards.value].sort((a, b) => a.rating.rate - b.rating.rate);
  } else if (order === 'desc') {
    cards.value = [...cards.value].sort((a, b) => b.rating.rate - a.rating.rate);
  } else {
    getLocalCards(); // Восстанавливаем начальный порядок карточек
  }
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

.sorting-panel {
  display: flex;
  gap: 5px;
  justify-content: center;
  padding-bottom: 10px;
  width: 100%;
  margin: 10px;

  .sort-btn {
    position: relative;
    padding: 5px;
    border-radius: 10px;
    transition: background-color 0.3s ease;

    &:hover {
      background-color: #e0e0e0;
    }

    .btn-text {
      position: absolute;
      /* Исправлено на absolute для точного позиционирования */
      top: 50%;
      left: 50%;
      /* Центрирование текста */
      transform: translate(-50%, 100%);
      /* Точная центровка */
      background-color: rgba(0, 0, 0, 0.7);
      color: white;
      padding: 5px 10px;
      border-radius: 5px;
      white-space: nowrap;
      opacity: 0;
      visibility: hidden;
      transition: opacity 0.9s, visibility 0.2s;
      z-index: 1;
      /* Поверхность текста над кнопкой */
    }

    &:hover .btn-text {
      opacity: 1;
      visibility: visible;
    }
  }
}
</style>
