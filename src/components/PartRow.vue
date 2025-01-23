<template>
  <tr>
    <td class="cell index-cell">{{ index }}</td>
    <td><input v-model="part.name" type="text" class="cell input-cell" /></td>
    <td class="cell">{{ part.price }}</td>
    <td>
      <button @click="decreaseQuantity" class="btn btn-sm">-</button>
      <input v-model.number="part.quantity" type="number" min="1" class="cell quantity-input" @input="updateCost" />
      <button @click="increaseQuantity" class="btn btn-sm">+</button>
    </td>
    <td class="cell cost-cell">{{ part.cost }}</td>
    <td>
      <button class="btn btn-danger" @click="confirmDelete">Удалить</button>
      <button class="btn btn-secondary" @click="$emit('add', part.id)">Добавить</button>
    </td>
  </tr>
</template>

<script lang="ts">
import { defineComponent, PropType } from 'vue';

export default defineComponent({
  name: 'PartRow',
  props: {
    part: {
      type: Object as PropType<{ id: number, name: string, price: number, quantity: number, cost: number, children: any[], index: string, parent?: any }>,
      required: true
    },
    index: {
      type: String,
      required: true
    }
  },
  methods: {
    increaseQuantity() {
      this.part.quantity++;
      this.updateCost();
    },
    decreaseQuantity() {
      if (this.part.quantity > 1) {
        this.part.quantity--;
        this.updateCost();
      }
    },
    updateCost() {
      this.part.cost = this.part.price * this.part.quantity;
      this.updateParentPrice();
    },
    updateParentPrice() {
      if (this.part.parent) {
        this.part.parent.price = this.part.parent.children.reduce((sum: number, child: any) => sum + child.cost, 0);
        this.part.parent.cost = this.part.parent.price * this.part.parent.quantity;
        if (this.part.parent.parent) {
          this.part.parent.updateParentPrice();
        }
      }
    },
    confirmDelete() {
      if (confirm('Вы уверены, что хотите удалить этот элемент?')) {
        this.$emit('delete', this.part.id);
      }
    }
  }
});
</script>

<style scoped>
.cell {
  border: 1px solid #ccc;
  padding: 5px;
  text-align: center;
}

.input-cell {
  width: 100px;
  text-align: center;
  border: 1px solid #999;
  border-radius: 4px;
  padding: 3px;
}

.quantity-input {
  width: 50px;
  text-align: center;
  border: 1px solid #999;
  border-radius: 4px;
  padding: 3px;
}

.cost-cell {
  background-color: #f8f8f8;
  font-weight: bold;
}

.btn-sm {
  padding: 2px 6px;
  font-size: 14px;
  margin: 0 3px;
  border: 1px solid #999;
  border-radius: 4px;
  background-color: #eee;
  cursor: pointer;
}
</style>
