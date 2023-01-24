
<template>
  <div class="button-holder">
    <button :class="Enemy.health > 0 ? 'hidden' : ' '" class="start-button" @click="startGame">Start</button>
  </div>
  <div class="content" :class="Enemy.health <= 0 ? 'hidden' : ' '">
    <div class="game">
      <div v-for="category in categories" :key="category.id" @drop="onDrop($event, category.id)" class="droppable"
        @dragover.prevent @dragenter.prevent>
        <div v-if="category.title === 'Enemy'" class="hero-item">
          <div class="card">
            <img :src="Enemy.img" alt="" class="hero-img">
            <progress id="file" max="30" :value=Enemy.health></progress> <span>{{ Enemy.health }} HP</span>
          </div>
        </div>
        <div class="item-holder">
          <div v-for="item in items.filter(x => x.categoryId === category.id)" @dragstart="onDragStart($event, item)"
            class="draggable" draggable="true" :class="category.title === 'Enemy' ? 'disabled' : ''">
            <img :src="item.img" alt="">
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'

const Enemy = ref({
  health: 0,
  img: "https://img.icons8.com/external-kosonicon-lineal-color-kosonicon/512/null/external-devil-halloween-kosonicon-lineal-color-kosonicon.png"
})

const items = ref([
  {
    id: 0,
    title: 'Hero',
    categoryId: 0,
    img: '',
    attack: 0,
  },
])

const categories = ref([
  {
    id: 0,
    title: 'Enemy'
  },
  {
    id: 1,
    title: 'Cards'
  }
])


const options = {
  method: 'GET',
  url: 'https://omgvamp-hearthstone-v1.p.rapidapi.com/cards/sets/Naxxramas',
  headers: {
    'X-RapidAPI-Key': 'c8e3b16864msha98c129da7f6dd9p1eb58ejsnd746187ba648',
    'X-RapidAPI-Host': 'omgvamp-hearthstone-v1.p.rapidapi.com'
  }
}

function onDragStart(e: DragEvent, item) {
  if (e.dataTransfer !== null) {
    e.dataTransfer.dropEffect = 'move'
    e.dataTransfer.effectAllowed = 'move'
    e.dataTransfer.setData('itemId', item.id.toString())
  }
}

function onDrop(e: DragEvent, categoryId) {
  if (e.dataTransfer !== null) {
    const itemId = parseInt(e.dataTransfer.getData('itemId'))

    items.value = items.value.map(x => {
      if (x.id === itemId && categoryId !== x.categoryId) {
        Enemy.value.health -= x.attack
      }
      return x
    })
  }
}

function startGame() {
  Enemy.value.health = 30
  items.value.splice(0)
  uploadCards()
}

function uploadCards() {
  axios.request(options).then(({ data }) => {
    const list: any[] = []

    data.forEach((item) => {
      if (item.type === 'Minion') {
        item.categoryId = 1
        item.title = item.name
        item.id = item.dbfId
        list.push(item)
      }
    })

    while (items.value.length !== 3) {
      const cardItem = list[Math.floor(Math.random() * list.length)]
      if (cardItem.img) {
        items.value.push(cardItem)
      }
    }
  }).catch(error => {
    console.error(error);
  })
}
</script>

<style scoped>
.card {
  background: rgb(59, 197, 231);
  display: flex;
  flex-direction: column;
  justify-content: center;
  border-radius: 20%;
  padding: 15px;
  align-items: center;
}

.hero-item {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.hero-img {
  width: 100px;
}

.disabled {
  display: none;
}

.droppable {
  padding: 15px;
  border-radius: 45px;
  background: #2c3e50;
  margin: 50px;
}

.droppable span {
  /* color: white; */
  text-align: center;
}

/* .draggable {
  background: white;
  padding: 5px;
  border-radius: 5px;
  margin-bottom: 5px;
} */

.draggable h5 {
  margin: 0;
}

.item-holder {

  display: flex;
  justify-content: space-around;
}

.start-button {
  background-color: #4CAF50;
  /* Green */
  border: none;
  color: white;
  padding: 16px 32px;
  text-align: center;
  text-decoration: none;
  display: inline-block;
  font-size: 16px;
  margin: 4px 2px;
  -webkit-transition-duration: 0.2s;
  /* Safari */
  transition-duration: 0.2s;
  cursor: pointer;
  background-color: white;
  color: black;
  border: 2px solid #4CAF50;
  border-radius: 25px;
}

.start-button:hover {
  background-color: #4CAF50;
  color: white;
}

.button-holder {
  display: flex;
  align-items: center;
  justify-content: center;
}

.content {
  margin-top: 25px;
  height: 550px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.game {
  background: rgba(23, 137, 3, 0.2);
  width: 50%;
  height: 100%;
  border-radius: 75px;
}

img {
  width: 150px;
}

.hidden {
  opacity: 0;
  visibility: hidden;
  transition: 0.4s;
}
</style>

