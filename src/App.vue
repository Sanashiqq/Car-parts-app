<template>
  <main>
    <button class="btn btn-primary" @click="exportToExcel">Экспорт в Excel</button>

    <table class="table">
      <thead>
        <tr>
          <th>Номер</th>
          <th>Деталь</th>
          <th>Цена</th>
          <th>Количество</th>
          <th>Стоимость</th>
          <th>Действия</th>
        </tr>
      </thead>
      <tbody>
        <PartRow
          v-for="(part, index) in getPartsWithIndexes(parts)"
          :key="part.id"
          :part="part"
          :index="part.index"
          :paddingLeft="0"
          @delete="deletePart"
          @add="showAddPartDialog"
          @updatePrice="updatePartPrice"
        />
      </tbody>
    </table>

    <div v-if="showDialog" class="modal">
      <div class="modal-content">
        <h3>Добавить новую деталь</h3>
        <label>Название:</label>
        <input v-model="newPart.name" type="text" />
        <label>Цена:</label>
        <input v-model.number="newPart.price" type="number" min="0" />
        <button class="btn btn-primary" @click="addPart">Добавить</button>
        <button class="btn btn-secondary" @click="closeDialog">Отмена</button>
      </div>
    </div>
  </main>
</template>

<script lang="ts">
import { defineComponent, reactive, ref } from 'vue';
import PartRow from './components/PartRow.vue'; // импортируем рекурсивный компонент
import * as XLSX from 'xlsx';

export default defineComponent({
  name: 'App',
  components: {
    PartRow
  },
  setup() {
    const parts = reactive([
      {
        id: 1,
        name: 'Кузов',
        cost: 0,
        price: 0,
        quantity: 1,
        children: reactive([
          {
            id: 2,
            name: 'Двери',
            cost: 0,
            price: 0,
            quantity: 3,
            children: reactive([
              { id: 3, name: 'Замок', cost: 0, price: 5000, quantity: 4, children: reactive([]) },
              { id: 4, name: 'Ручки', cost: 0, price: 6000, quantity: 6, children: reactive([]) }
            ])
          }
        ])
      },
      {
        id: 5,
        name: 'Двигатель',
        cost: 0,
        price: 0,
        quantity: 1,
        children: reactive([
          { id: 6, name: 'Поршни', cost: 0, price: 10000, quantity: 5, children: reactive([]) },
          { id: 7, name: 'Кольца', cost: 0, price: 2000, quantity: 3, children: reactive([]) }
        ])
      }
    ]);

    const showDialog = ref(false);
    const newPart = reactive({ name: '', price: 0, parentId: null });
    
    const recalculatePrices = () => {
      const calculateCost = (part: any) => {
    // Если у части есть дети, мы сначала считаем их стоимость
    if (part.children && part.children.length > 0) {
      part.price = part.children.reduce((sum: number, child: any) => sum + calculateCost(child), 0);
    }
    // Вычисляем общую стоимость
    console.log(part)
    part.cost = part.price * (part.quantity || 1);
    return part.cost;
  };
  // Для начала пересчитываем все части начиная с самой верхней
  parts.forEach(calculateCost);
};

    const deletePart = (id: number) => {
      const removePart = (partsList: any[]) => {
        return partsList.filter(part => {
          if (part.id === id) return false;
          if (part.children) part.children = removePart(part.children);
          return true;
        });
      };
      parts.splice(0, parts.length, ...removePart(parts));
      recalculatePrices();
    };

    const showAddPartDialog = (parentId: number) => {
      newPart.name = '';
      newPart.price = 0;
      newPart.parentId = parentId;
      showDialog.value = true;
    };

    const closeDialog = () => {
      showDialog.value = false;
    };

    const resetNewPart = () => {
      newPart.name = '';
      newPart.price = 0;
      newPart.parentId = null;
      showDialog.value = false; 
    };

    const addPart = () => {
      if (!newPart.name || newPart.price <= 0) return;

      const findAndAdd = (partsList: any[]) => {
        for (let part of partsList) {
          if (part.id === newPart.parentId) {
            if (!part.children) part.children = reactive([]);

            const newChild = {
              id: Date.now(),
              name: newPart.name,
              price: newPart.price,
              quantity: 1,
              children: reactive([]),
            };

            part.children.push(newChild);
            return true;
          }
          if (part.children && findAndAdd(part.children)) return true;
        }
        return false;
      };

      if (findAndAdd(parts)) {
        recalculatePrices();
        resetNewPart(); 
      }
    };

    const getPartsWithIndexes = (parts: any[], index: string = '') => {
      let result: any[] = [];

      const processPart = (part: any, parentName: string | null = null, index: string = '') => {
        part.index = index; // добавляем индекс в объект детали
        result.push(part);

        if (part.children) {
          part.children.forEach((child: any, i: number) => {
            processPart(child, part.name, `${index}.${i + 1}`);
          });
        }
      };

      parts.forEach((part, i) => processPart(part, null, `${i + 1}`));
      return result;
    };

    const exportToExcel = () => {
  // Пересчитываем все стоимости перед экспортом
  recalculatePrices();

  // Форматируем данные для экспорта
  const formatData = (parts: any[]) => {
    let result: any[] = [];
    const processPart = (part: any, parentName: string | null = null, index: string = '') => {
      const partData = {
        Номер: index, //номер отдельно
        Название: part.name, //в отдельном столбце
        Цена: part.price,
        Количество: part.quantity,
        Стоимость: part.cost,
        Родитель: parentName || '',
      };
      result.push(partData);

      if (part.children) {
        part.children.forEach((child: any, i: number) => {
          processPart(child, part.name, `${index}.${i + 1}`);
        });
      }
    };

    parts.forEach((part, i) => processPart(part, null, `${i + 1}`));
    return result;
  };

  // Получаем актуальные данные для экспорта после пересчета
  const data = formatData(parts);
  const ws = XLSX.utils.json_to_sheet(data);
  const wb = XLSX.utils.book_new();
  XLSX.utils.book_append_sheet(wb, ws, 'Таблица деталей');
  XLSX.writeFile(wb, 'car_parts.xlsx');
};


    const updatePartPrice = (part: any, newPrice: number) => {
      part.price = newPrice;  // Обновляем цену детальки
      recalculatePrices();  // Пересчитываем цены для всех родительских деталей
    };

    recalculatePrices();

    return { parts, recalculatePrices, deletePart, showAddPartDialog, addPart, closeDialog, showDialog, newPart, exportToExcel, getPartsWithIndexes, updatePartPrice };
  }
});
</script>

<style scoped>
.child-row {
  background-color: #f9f9f9;
  padding-left: 20px;
}
.sub-child-row {
  background-color: #e9e9e9;
  padding-left: 40px;
}
.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal-content {
  background: white;
  padding: 20px;
  border-radius: 5px;
  text-align: center;
}
</style>
