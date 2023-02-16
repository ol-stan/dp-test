<template>
  <div class="todo">
    <div class="todo-list">
      <div class="todo-list-actions">
        <input type="text" v-model="search" placeholder="Search in title" />
        <select v-model="statusFilter">
          <option value="" :selected="true" disabled>Filter by Status</option>
          <option v-for="item in statuses" :key="item.id" :value="item">{{ item }}</option>
        </select>
        <select v-model="userIdFilter">
          <option value="" :selected="true" disabled>Filter by UserId</option>
          <option v-for="item in usersIds" :key="item.id" :value="item">{{ item }}</option>
          <option value="all">All users</option>
        </select>
      </div>
      <div class="todo-list-caption todo-grid-row">
        <span class="todo-list-item" @click="sortBy('userId')">User Id</span>
        <span class="todo-list-item" @click="sortBy('id')">Id</span>
        <span class="todo-list-item" @click="sortBy('title')">Title</span>
        <span class="todo-list-item">Completed</span>
        <span class="todo-list-item">Favorites</span>
      </div>
      <template v-if="filteredItems.length">
        <div class="todo-list-row todo-grid-row" v-for="item in filteredItems" :key="item.id">
          <span class="todo-list-item">{{ item.userId }}</span>
          <span class="todo-list-item">{{ item.id }}</span>
          <span class="todo-list-item todo-list-item--left">{{ item.title }}</span>
          <span class="todo-list-item">{{ item.completed }}</span>
          <span class="todo-list-item todo-list-item--favorite" :id="item.id" @click="toggleFavorite">
            {{ item.favorite }}
          </span>
        </div>
      </template>
      <div v-else class="todo-list-noitem">
        Sorry, but no items were found for your search criteria
      </div>
    </div>
    <div class="todo-add">
      <div class="todo-add-title">Create ToDo</div>
      <input v-model="addTodoUserId" class="todo-add-input" type="text" placeholder="User ID">
      <span class="todo-add-input-message" v-if="messageUserId">{{ messageUserId }}</span>
      <input v-model="addTodoTitle" class="todo-add-input" type="text" placeholder="Title">
      <button class="todo-add-button" :disabled="isLoading" @click="addTodo">Add</button>
    </div>
  </div>
</template>

<script>
import { computed, onMounted, reactive, ref } from 'vue'
import axios from 'axios'

export default {
  name: 'TodoList',
  props: {
    items: { type: Object || null, default: null },
  },
  setup(props) {
    const search = ref('')
    const userIdFilter = ref('')
    const statusFilter = ref('')
    const sortKey = ref('Id')
    const reverse = ref(false)
    const statuses = ['All', 'Completed', 'Uncompleted', 'Favorites']
    const favorites = reactive({ ids: [] })
    const newItems = reactive({ list: [] })
    const addTodoUserId = ref('')
    const addTodoTitle = ref('')
    const messageUserId = ref('')
    const isLoading = ref(false)

    onMounted(() => {
      favorites.value = JSON.parse(localStorage.getItem('favorites')) || []
    })

    const toggleFavorite = ($event) => {
      const id = $event.target.id
      if (favorites.value.includes(id)) {
        favorites.value = favorites.value.filter(favoriteId => favoriteId !== id)
      } else {
        favorites.value.push(id)
      }
      localStorage.setItem('favorites', JSON.stringify(favorites.value))
    }

    const filteredItems = computed(() => {
      let items = props.items || []

      if (newItems.list.length) {
        items = [...newItems.list, ...items]
      }

      items = items.map(item => {
        return { ...item, favorite: favorites.value?.includes(String(item.id)) ? 'remove' : 'add' }
      })

      if (search.value) {
        items = items.filter(item => {
          return item.title.toLowerCase().includes(search.value.toLowerCase())
        })
      }

      if (userIdFilter.value !== '' && userIdFilter.value !== 'all') {
        items = items.filter((item) => item.userId === userIdFilter.value)
      }

      if (statusFilter.value !== '') {
        if (statusFilter.value === "Completed") {
          items = items.filter((item) => item.completed === true)
        }
        if (statusFilter.value === "Uncompleted") {
          items = items.filter((item) => item.completed === false)
        }
        if (statusFilter.value === "Favorites") {
          items = items.filter((item) => item.favorite === "remove")
        }
      }

      const order = reverse.value ? -1 : 1;
      items = items.sort((a, b) => a[sortKey.value] > b[sortKey.value] ? order : -order)

      return items
    })

    const usersIds = computed(() => {
      const userIdSet = new Set(props.items.map(item => item.userId))

      return Array.from(userIdSet)
    })

    const addTodo = async () => {
      if (!addTodoUserId.value || !addTodoTitle.value) return
      messageUserId.value = ''
      if (!(/^\d+$/).test(addTodoUserId.value)) {
        messageUserId.value = 'Use only whole numbers'
        return addTodoUserId.value = ''
      }
      if (!usersIds.value.includes(Number(addTodoUserId.value))) {
        messageUserId.value = 'User not found, check and try again'
        return addTodoUserId.value
      }

      const newItem = {
        id: filteredItems.value.length + 1,
        userId: addTodoUserId.value,
        title: addTodoTitle.value,
        completed: false
      }

      isLoading.value = true

      await axios.post('https://jsonplaceholder.typicode.com/todos', newItem)
        .then(response => {
          if ((/^2\d{2}$/).test(response.status)) {
            isLoading.value = false
            newItems.list.push(newItem)
            addTodoUserId.value = ''
            addTodoTitle.value = ''
          } else {
            console.log('Something went wrong :(')
          }
        })
        .catch(error => {
          console.log(error)
        })
    }

    const sortBy = (key) => {
      if (sortKey.value === key) {
        reverse.value = !reverse.value
      } else {
        sortKey.value = key
        reverse.value = false
      }
    }

    return {
      search,
      userIdFilter,
      statusFilter,
      statuses,
      usersIds,
      favorites,
      filteredItems,
      addTodoUserId,
      addTodoTitle,
      messageUserId,
      isLoading,
      addTodo,
      sortBy,
      toggleFavorite
    }
  }
}
</script>

<style scoped lang="scss">
.todo {
  display: grid;
  grid-template-columns: 3fr 1fr;
  grid-column-gap: 15px;
  padding: 10px 10px 20px;
  background: #C4C4C4;
  border-radius: 0 0 5px 5px;

  @media screen and (max-width: 767px) {
    grid-template-columns: 1fr;
    grid-row-gap: 10px;
  }

  @media screen and (max-width: 320px) {
    padding: 10px 0 20px;
  }
}

.todo-list {
  &-actions {

    @media screen and (max-width: 425px) {
      padding: 0 10px;
    }

    input,
    select {
      margin: 0 10px 10px 0;
      padding: 8px 12px;
      font-size: 14px;
      color: #353535;
      border: none;
      border-radius: 5px;

      @media screen and (max-width: 425px) {
        width: 100%;
      }
    }

    input::placeholder {
      color: #353535;
    }
  }

  &-row {
    border: 1px solid #A5A5A5;
    border-block: none;

    &:last-of-type {
      border-bottom: 1px solid #A5A5A5;
    }
  }

  &-item {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    text-align: center;
    padding: 4px;
    color: #353535;
    border: 1px solid #A5A5A5;
    overflow: hidden;

    @media screen and (max-width: 425px) {
      font-size: 14px;
    }

    @media screen and (max-width: 320px) {
      font-size: 13px;
    }

    &--left {
      justify-content: left;
      text-align: left;
    }

    &--favorite {
      text-decoration: underline;
      cursor: pointer;
      transition: all 0.3s ease;

      &:hover {
        text-decoration: none;
      }
    }
  }

  &-caption {
    background: #A5A5A5;
    border: 1px solid #828282;
    border-bottom: none;
    border-radius: 5px 5px 0 0;

    & .todo-list-item {
      font-size: 13px;
      font-weight: 700;

      @media screen and (max-width: 425px) {
        font-size: 12px;
      }
    }
  }

  &-noitem {
    padding: 30px 20px;
    text-align: center;
    color: rgb(82, 10, 10);
  }
}

.todo-add {
  position: sticky;
  top: 16px;
  display: flex;
  flex-direction: column;
  align-self: start;
  padding: 10px;
  background: #A5A5A5;
  border-radius: 5px;

  @media screen and (max-width: 767px) {
    position: static;
    order: -1;
  }

  &-title {
    margin: 0 0 15px 0;
    font-size: 17px;
    line-height: 1.24;
    font-weight: 700;
    color: #353535;
  }

  &-input {
    margin: 0 0 15px 0;
    padding: 10px;
    font-size: 16px;
    line-height: 1.24;
    letter-spacing: -0.025em;
    color: #353535;
    background: #FFFFFF;
    border: none;
    border-radius: 5px;
  }

  &-input-message {
    font-size: 13px;
    color: red;
    margin: -5px 0 15px 0;
  }

  &-button {
    margin: 5px 0 0 0;
    padding: 8px 30px;
    letter-spacing: -0.025em;
    font-size: 17px;
    font-weight: 600;
    line-height: 1.24;
    color: #519945;
    border: 2px solid #519945;
    background: #ffffff;
    border-radius: 5px;
    transition: all 0.3s ease;
    cursor: pointer;

    &:hover {
      color: #ffffff;
      background: #519945;
      border-color: transparent;
    }
  }
}

.todo-grid-row {
  display: grid;
  grid-template-columns: repeat(2, minmax(2rem, 1fr)) minmax(4.1rem, 10fr) repeat(2, minmax(3.9rem, 2fr));
}
</style>