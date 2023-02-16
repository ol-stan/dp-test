<template>
  <AppHeader />
  <main class="main">
    <router-view v-slot="slotProps">
      <transition name="route" mode="out-in">
        <component :is="slotProps.Component"></component>
      </transition>
    </router-view>
  </main>
  <AppFooter />
</template>

<script>
import AppHeader from './components/AppHeader.vue'
import AppFooter from './components/AppFooter.vue'

export default {
  name: 'App',
  components: {
    AppHeader, AppFooter
  }
}
</script>

<style lang="scss">
html,
body {
  margin: 0;
  padding: 0;
}

body {
  font-family: 'Roboto', Arial, Helvetica, sans-serif;
}

* {
  box-sizing: border-box;
}

.main {
  min-height: calc(100vh - 330px);
  padding: 0 20px;
  background: #828282;

  @media screen and (max-width: 375px) {
    padding: 0 10px;
  }
}

.route-enter-from,
.route-leave-to {
  opacity: 0;
}

.route-enter-active {
  transition: all 0.3s ease-out;
}

.route-leave-active {
  transition: all 0.3s ease-in;
}

.route-enter-to,
.route-leave-from {
  opacity: 1;
}
</style>
