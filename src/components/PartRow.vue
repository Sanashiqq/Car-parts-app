<template>
  <tr>
    <td class="cell index-cell">{{ index }}</td>
    <td><input v-model="part.name" type="text" class="cell input-cell" /></td>
    <td class="cell">{{ computedPrice }}</td>
    <td>
      <button @click="decreaseQuantity" class="btn btn-sm">-</button>
      <input v-model.number="part.quantity" type="number" min="1" class="cell quantity-input" />
      <button @click="increaseQuantity" class="btn btn-sm">+</button>
    </td>
    <td class="cell cost-cell">{{ computedCost }}</td>
    <td>
      <button class="btn btn-danger" @click="confirmDelete">Удалить</button>
      <button class="btn btn-secondary" @click="$emit('add', part.id)">Добавить</button>
    </td>
  </tr>
</template>

<script lang="ts">
import { defineComponent, PropType, computed, watch, onMounted, nextTick } from 'vue';

export default defineComponent({
  name: 'PartRow',
  props: {
    part: {
      type: Object as PropType<{ 
        id: number, 
        name: string, 
        price: number, 
        quantity: number, 
        cost: number, 
        children: any[], 
        index: string, 
        parent?: any 
      }>,
      required: true
    },
    index: {
      type: String,
      required: true
    }
  },
  setup(props) {
    const computedPrice = computed(() => {
  // Начальный расчет цены на основе детей, если они есть
  const total = props.part.children.length > 0 
    ? props.part.children.reduce((sum, child) => {
        console.log(`Дочерняя деталь: ${child.name}, Стоимость: ${child.cost}, Количество: ${child.quantity} `);
        return sum + child.cost;
      }, 0) 
    : props.part.price;

  // Отсроченный вывод после обновления DOM
  nextTick(() => {
    console.log(`Пересчитанная цена для ${props.part.name}:`, total);
  });

  return total;
});


    const computedCost = computed(() => computedPrice.value * props.part.quantity);

    watch(computedCost, () => {
      props.part.cost = computedCost.value;
      updateParentCost();
    });

    watch(() => props.part.children, () => {
      updateParentCost();
    }, { deep: true });

    const updateParentCost = () => {
      if (props.part.parent) {
        props.part.parent.cost = props.part.parent.children.reduce((sum: any, child: { cost: any; }) => sum + child.cost, 0);
        updateParentCost();
      }
    };

    onMounted(() => {
      props.part.cost = computedCost.value;
      updateParentCost();
    });

    return { computedPrice, computedCost };
  },
  methods: {
    increaseQuantity() {
      this.part.quantity++;
    },
    decreaseQuantity() {
      if (this.part.quantity > 1) {
        this.part.quantity--;
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
