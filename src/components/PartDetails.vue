<template>
  <tr>
    <td>{{ part.name }}</td>
    <td>{{ part.price }}</td>
    <td>
      <input type="number" v-model="part.quantity" min="0" @input="updateCost" />
    </td>
    <td>{{ part.cost }}</td>
    <td>
      <button class="btn btn-danger" @click="$emit('delete', part.id)">Удалить</button>
      <button class="btn btn-success" @click="$emit('add', part.id)">Добавить</button>
    </td>
  </tr>
</template>

<script lang="ts">
import { defineComponent, watch } from 'vue';

export default defineComponent({
  name: 'PartDetails',
  props: {
    part: {
      type: Object,
      required: true,
    },
  },
  emits: ['delete', 'add'],
  methods: {
    updateCost() {
      this.part.cost = this.part.price * this.part.quantity;
    },
  },
});
</script>
