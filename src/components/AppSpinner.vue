<template>
  <div class="spinner-wrapper">
    <span class="spinner"></span>
  </div>
</template>

<script>
export default {
  name: 'AppSpinner'
}
</script>

<style scoped lang="scss">
.spinner-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 10px;
  display: flex;
  justify-content: center;
  align-items: center;
  box-sizing: border-box;
  overflow: hidden;
}

.spinner {
  width: 80px;
  height: 80px;
  margin: 0;
  background: transparent;
  border-top: 5px solid #ffffff;
  border-right: 5px solid transparent;
  border-radius: 50%;
  animation: 1s spin linear infinite;
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
}
</style>