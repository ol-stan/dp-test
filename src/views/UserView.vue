<template>
  <div class="user">
    <div class="user-block">
      <div class="block-title">Personal data</div>
      <div v-if="user.isLoading" class="block-loading">
        <AppSpinner />
      </div>
      <ul v-else class="user-data-list">
        <li class="user-data-item" v-for="(item, key) in user.data" :key="key">
          {{ key }}
          <UserDataItem :item="item" />
        </li>
      </ul>
    </div>

    <div class="todo-block">
      <div class="block-title">ToDo</div>
      <div v-if="todos.isLoading" class="block-loading">
        <AppSpinner />
      </div>
      <TodoList v-else :items="todos.list" />
    </div>
  </div>
</template>

<script>
import UserDataItem from '../components/UserDataItem.vue'
import TodoList from '../components/TodoList.vue'
import AppSpinner from '../components/AppSpinner.vue'

import { useRoute } from 'vue-router'
import { onMounted, reactive } from 'vue'
import axios from 'axios'

export default {
  name: 'UserView',
  components: { AppSpinner, UserDataItem, TodoList },
  setup() {
    const route = useRoute()
    const currentUserId = route.params.id
    const user = reactive({ data: [], isLoading: true })
    const todos = reactive({ list: [], isLoading: true })

    onMounted(async () => {
      await axios.get(`https://jsonplaceholder.typicode.com/users/${currentUserId}`)
        .then(res => {
          user.isLoading = false
          user.data = res.data
        })
        .catch(error => {
          console.log(error)
        })
      await axios.get('https://jsonplaceholder.typicode.com/todos')
        .then(res => {
          todos.isLoading = false
          todos.list = res.data
        })
        .catch(error => {
          console.log(error)
        })
    })

    return {
      user,
      todos
    }
  }
}
</script>

<style scoped lang="scss">
.user {
  padding: 30px 0;
}

.block-title {
  margin: 0;
  padding: 15px 8px;
  font-size: 17px;
  font-weight: 700;
  color: #353535;
  background: #A5A5A5;
  border-radius: 5px 5px 0 0;
}

.user-data {
  &-list {
    list-style-type: none;
    margin: 0 0 32px 0;
    padding: 10px;
    background: #C4C4C4;
    border-radius: 0 0 5px 5px;
  }


  &-item {
    display: flex;
    flex-flow: row nowrap;
    align-items: center;
    justify-content: space-between;
    margin: 0 0 5px 0;
    padding: 0 0 0 8px;
    color: #353535;
    border: 1px solid #A5A5A5;
    border-radius: 5px;
    background: #A5A5A5;
    overflow: hidden;

    & span {
      max-width: 85%;
      width: 100%;
      padding: 4px 8px;
      background: #ffffff;
      border-radius: 0 5px 5px 0;

      @media screen and (max-width: 767px) {
        max-width: 75%;
      }

      @media screen and (max-width: 320px) {
        max-width: 65%;
      }

    }
  }
}

.block-loading {
  width: 100%;
  height: 310px;
  margin: 0 0 32px 0;
  background: #C4C4C4;
  border-radius: 0 0 5px 5px;
}
</style>