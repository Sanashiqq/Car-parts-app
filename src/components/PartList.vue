<template>
  <table class="table">
    <thead>
      <tr>
        <th>Деталь</th>
        <th>Цена</th>
        <th>Количество</th>
        <th>Стоимость</th>
        <th>Действия</th>
      </tr>
    </thead>
    <tbody>
      <PartDetails
        v-for="part in parts"
        :key="part.id"
        :part="part"
        @delete="deletePart"
        @add="addPart"
      />
    </tbody>
  </table>
</template>

<script lang="ts">
import { defineComponent } from 'vue';
import PartDetails from './PartDetails.vue';

export default defineComponent({
  name: 'PartList',
  components: { PartDetails },
  data() {
    return {
      parts: [
        { id: 1, name: 'Кузов', price: 11000, quantity: 1, cost: 11000 },
        { id: 2, name: 'Двери', price: 11000, quantity: 3, cost: 33000 },
        { id: 3, name: 'Замок', price: 5000, quantity: 4, cost: 20000 },
        { id: 4, name: 'Ручки', price: 6000, quantity: 6, cost: 36000 },
      ],
    };
  },
  methods: {
    deletePart(id: number) {
      this.parts = this.parts.filter((part) => part.id !== id);
    },
    addPart(id: number) {
      const newPart = this.parts.find((part) => part.id === id);
      if (newPart) {
        const copy = { ...newPart, id: Date.now() };
        this.parts.push(copy);
      }
    },
  },
});
</script>
