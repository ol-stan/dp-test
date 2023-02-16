<template>
  <span>
    <template v-if="typeof item === 'object' && item !== null">
      <span v-for="(value, key) in item" :key="key">
        {{ key }}:
        <UserDataItem :item="value" :index="key" />&nbsp;
      </span>
    </template>
    <template v-else>
      {{ item }}
    </template>
  </span>
</template>

<script>
export default {
  name: 'UserDataItem',
  props: {
    item: { type: [Object, Number, String, null], default: null },
    index: { type: String, default: '' }
  }
}
</script>