<template>
  <div class="home">
    <div class="form">
      <div class="form-header">description</div>
      <div class="form-body">
        <p class="form-description">description</p>
        <input class="form-input" type="text" id="username" title="Only letters" placeholder="Username"
          autocomplete="off" :value="form.username" @input="validate" />
        <input class="form-input" type="text" id="phone" title="Numbers or symbols" placeholder="Phone number"
          autocomplete="off" :value="form.phone" @input="validate" />
        <button class="form-button" :disabled="isLoading" type="submit" @click="auth">Login</button>
        <div class="message" v-if="message">{{ message }}</div>
      </div>
    </div>
  </div>
</template>

<script>
import { onMounted, reactive, ref } from 'vue'
import axios from 'axios'
import { useRouter } from 'vue-router'

export default {
  name: 'HomeView',
  setup() {
    const form = reactive({ username: '', phone: '' })
    const users = reactive({ data: [] })
    const isLoading = ref(false)
    const message = ref('')
    const router = useRouter()

    const validate = (event) => {
      const check = {
        username: /^[A-z\s]*$/,
        phone: /^[D\s!@#$%^&*()-_=+,.?":{}|<>]*$/
      }
      if (check[event.target.id].test(event.target.value)) {
        form[event.target.id] = event.target.value
      } else {
        event.target.value = form[event.target.id]
      }
    }

    onMounted(async () => {
      isLoading.value = true
      await axios.get('https://jsonplaceholder.typicode.com/users')
        .then(res => {
          users.data = res.data
          isLoading.value = false
        })
        .catch(error => {
          console.log(error)
        })
    })

    const auth = () => {
      const isLogined = ref(false)
      const formPhone = form.phone.replace(/\D/g, '')
      users.data.forEach(el => {
        if (el.username.replace(/[^A-z]/g, '') === form.username &&
          el.phone.replace(/(\D)([A-z].*)/, '$1').replace(/\D/g, '') === formPhone) {
          router.push({ name: 'user', params: { id: el.id } })
          return isLogined.value = true
        }
      })
      if (!isLogined.value) {
        message.value = 'Login error'
      }
    }

    return {
      form,
      users,
      isLoading,
      message,
      auth,
      validate
    }
  }
}
</script>

<style lang="scss">
.home {
  width: 100%;
  height: 100%;
  min-height: 460px;
}

.form {
  position: absolute;
  top: 20%;
  left: 50%;
  transform: translateX(-50%);
  width: 100%;
  max-width: 447px;
  background: #C4C4C4;
  border-radius: 5px;
  z-index: 5;
  overflow: hidden;

  &-header {
    padding: 15px 25px;
    color: #5F5F5F;
    font-size: 17px;
    line-height: 1.24;
    letter-spacing: -0.025em;
    text-align: center;
    background: #A5A5A5;
  }

  &-body {
    display: flex;
    flex-direction: column;
    padding: 15px 25px 30px;
  }

  &-description {
    margin: 0 0 14px 0;
    padding: 0;
    font-size: 15px;
    line-height: 1.4;
    letter-spacing: -0.025em;
    color: #5F5F5F;
  }

  &-input {
    margin: 0 0 20px 0;
    padding: 10px;
    font-size: 17px;
    font-weight: 600;
    line-height: 1.24;
    letter-spacing: -0.025em;
    color: #353535;
    background: #FFFFFF;
    border: none;
    border-radius: 5px;

    &::placeholder {
      font-weight: 400;
      color: #353535;
    }
  }

  &-button {
    margin: 5px 0 0 0;
    padding: 8px 30px;
    letter-spacing: -0.025em;
    font-size: 17px;
    font-weight: 600;
    line-height: 1.24;
    color: #FFFFFF;
    border: 2px solid transparent;
    background: #519945;
    border-radius: 5px;
    cursor: pointer;

    &:hover {
      color: #519945;
      background: #ffffff;
      border-color: #519945;
      transition: all 0.3s ease;
    }
  }
}

.message {
  margin: 10px 0;
  color: red
}
</style>
