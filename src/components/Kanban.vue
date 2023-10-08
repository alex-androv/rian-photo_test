<template>
  <div class="kanban-component">
    <div class="columns">
      <div class="column" v-for="(column, index) in columns" :key="index">
        <div :style="{ backgroundColor: column.color }" class="column__header">
          <h2>{{ column.name }} ({{ column.cards.length }})</h2>
          <button class="btn" @click="createCard(index)">Создать</button>
          <button class="btn" @click="sortCards(index)">Сортировать</button>
        </div>
        <draggable class="cards" v-model="column.cards" group="products" @change="updateSelected(index)">
          <template #item="{ element, index: i }">
            <div class="card" :style="{ backgroundColor: column.color }">
              <img :src="element.image" alt="Product Image">
              <div class="card-info">
                <p class="card-title"><b>Название:</b> {{ element.title }}</p>
                <p><b>Цена:</b> {{ element.price }}</p>
                <p><b>Категория:</b> {{ element.category }}</p>
                <p class="card-desc"><b>Описание:</b> {{ element.description }}</p>
                <p><b>Рейтинг:</b> {{ element.rating.rate }} ({{ element.rating.count }})</p>
                <button @click="editCard(index, i)">Редактировать</button>
                <button @click="deleteCard(index, i)">Удалить</button>
              </div>
            </div>
          </template>
        </draggable>
        <p v-if="!column.cards.length">Ничего не добавлено</p>
      </div>
    </div>
  </div>
</template>

<script>
  import axios from "axios";
  import draggable from "vuedraggable";

  export default {
    name: "KanbanComponent",
    components: {
      draggable,
    },
    data() {
      return {
        columns: [{
            name: "Необработанные",
            color: "lightgray",
            cards: [],
          },
          {
            name: "В работе",
            color: "lightblue",
            cards: [],
          },
          {
            name: "Завершенные",
            color: "lightgreen",
            cards: [],
          },
        ],
      };
    },
    mounted() {
      axios
        .get("https://fakestoreapi.com/products")
        .then((response) => {
          this.columns[0].cards = response.data;
        })
        .catch((error) => {
          console.error(error);
        });
    },
    methods: {
      createCard(columnIndex) {
        const title = prompt("Введите название товара");
        const price = prompt("Введите цену товара");
        const category = prompt("Введите категорию товара");
        const description = prompt("Введите описание товара");
        const rating = prompt("Введите рейтинг товара от 0 до 5");
        const image = prompt("Введите URL фото товара");
        if (
          title &&
          price &&
          category &&
          description &&
          rating &&
          image &&
          !isNaN(price) &&
          !isNaN(rating) &&
          rating >= 0 &&
          rating <= 5
        ) {
          const card = {
            title,
            price,
            category,
            description,
            rating: {
              rate: rating,
              count: 1,
            },
            image,
          };

          this.columns[columnIndex].cards.push(card);
        } else {
          alert("Данные введены некорректно. Попробуйсте снова.");
        }
      },
      deleteCard(columnIndex, cardIndex) {
        if (confirm("Вы уверены что хотите удалить товар")) {
          this.columns[columnIndex].cards.splice(cardIndex, 1);
        }
      },
      editCard(columnIndex, cardIndex) {
        const card = this.columns[columnIndex].cards[cardIndex];
        if (card) {
          const title = prompt("Введите новое название товара", card.title);
          const price = prompt("Введите новую цену товара", card.price);
          const category = prompt(
            "Введите новую категорию товара",
            card.category
          );
          const description = prompt(
            "Введите новое описание товараd",
            card.description
          );
          const rating = prompt(
            "Введите новый рейтинг товара от 0 до 5",
            card.rating.rate
          );
          const image = prompt("Введите новый URL фото товара", card.image);

          if (
            title &&
            price &&
            category &&
            description &&
            rating &&
            image &&
            !isNaN(price) &&
            !isNaN(rating) &&
            rating >= 0 &&
            rating <= 5
          ) {
            card.title = title;
            card.price = price;
            card.category = category;
            card.description = description;
            card.rating.rate = rating;
            card.image = image;
          } else {
            alert("Данные введены некорректно. Попробуйсте снова.");
          }
        } else {
          alert("Карта, которую вы пытаетесь изменить, не определена. Попробуйте еще раз.");
        }
      },
      sortCards(columnIndex) {
        const column = this.columns[columnIndex];
        column.cards.sort((a, b) => b.rating.rate - a.rating.rate);
      },
      updateSelected(index) {
        const list = this.lists[index];
        list.selected = [];
        list.items.forEach((item) => {
          if (item.checked) {
            for (let i = 0; i < item.quantity; i++) {
              list.selected.push(item);
            }
          }
        });
      },
    },
  };
</script>

<style>
  .kanban-component {
    margin: 0 300px;
  }

  .columns {
    display: flex;
  }

  .column {
    width: 33%;
    padding: 10px;
  }

  .column__header {
    padding: 20px 0;
    border-radius: 16px;
    margin-bottom: 10px;

  }

  .column h2 {
    text-align: center;
    color: white;
    margin-top: 0;
  }

  .cards {
    min-height: 100px;
  }

  .card {
    padding: 20px;
    cursor: pointer;
    border-radius: 16px;
    margin-bottom: 10px;
  }

  .card img {
    width: 100%;
    aspect-ratio: 1 / 1;
    object-fit: cover;
  }

  .card-info {
    background: #FFFFFF;
    padding: 10px 20px;
  }

  .card-title {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    margin: 0;

  }

  .card-desc {
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .card button {
    display: block;
    margin: 5px auto;
    background: #0371EC;
    border: 2px solid #0371EC;
    border-radius: 8px;
    font-family: 'Roboto';
    font-style: normal;
    font-weight: 500;
    font-size: 16px;
    line-height: 19px;
    text-align: center;
    color: #FFFFFF;
    padding: 8px 16px;
    cursor: pointer;
  }

  .card button:hover {
    color: #c5c8db;
    box-shadow: 0 7px 14px rgba(0, 0, 0, 0.18), 0 5px 5px rgba(0, 0, 0, 0.12);
  }

  .btn {
    background: #21afa3;
    border: 2px solid #21afa3;
    border-radius: 8px;
    font-family: 'Roboto';
    font-style: normal;
    font-weight: 500;
    font-size: 16px;
    line-height: 19px;
    text-align: center;
    color: #FFFFFF;
    padding: 8px 16px;
    margin: 5px;
    cursor: pointer;
    transition-duration: 100ms;
  }

  .btn:hover {
    color: #c5c8db;
    box-shadow: 0 7px 14px rgba(0, 0, 0, 0.18), 0 5px 5px rgba(0, 0, 0, 0.12);
  }

  @media screen and (max-width: 1800px) {

    .kanban-component {
      margin: 0 100px;
    }
  }

  @media screen and (max-width: 1100px) {
    .column h2 {
      font-size: 18px;
    }

    .kanban-component {
      margin: 0 50px;
    }
  }
</style>