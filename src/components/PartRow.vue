<template>
  <tr>
    <td>{{ part.name }}</td>
    <td>{{ part.price }}</td>
    <td>
      <input type="number" v-model="part.quantity" min="0" @input="$emit('update', part)" />
    </td>
    <td>{{ part.cost }}</td>
    <td>
      <button class="btn btn-danger" @click="$emit('delete', part.id)">Удалить</button>
      <button class="btn btn-success" @click="$emit('add', part.id)">Добавить</button>
    </td>
  </tr>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'PartRow',
  props: {
    part: Object,
  },
  emits: ['update', 'delete', 'add']
});
</script>
