<template>
  <div>
    <b-alert v-model="visibleLocal" variant="success" dismissible>{{ message }}</b-alert>
  </div>
</template>


<script>
export default {
  props: ['message', 'visible'],
  computed: {
    visibleLocal: {
      get() {
        return this.visible;
      },
      set(value) {
        this.$emit('update:visible', value); // Event изменения visible
      },
    },
  },
};
</script>
