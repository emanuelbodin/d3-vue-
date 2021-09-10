<template>
  <div class="legend-container">
    <div v-for="data in dataset.data" :key="data.id">
      <label>{{ data.name }}</label>
      <input
        type="checkbox"
        :id="data.id"
        :name="data.name"
        :checked="data.visible"
        @change="$emit('toggle', data.id)"
      />
    </div>
  </div>
</template>

<script>
export default {
  name: 'Legends',
  props: {
    dataset: {
      type: Object,
      required: true,
    },
  },
};
</script>

<style>
.legend-container {
  display: flex;
  justify-content: space-around;
}
</style>
