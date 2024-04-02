<script setup lang="ts">
const columns = [
  {
    key: 'title',
    label: 'Title',
    sortable: true
  }, {
    key: 'description',
    label: 'Description',
    sortable: true
  }, {
    key: 'price',
    label: 'Price',
    sortable: true
  }, {
    key: 'rating',
    label: 'Rating',
    sortable: true
  }, {
    key: 'brand',
    label: 'Brand',
    sortable: true
  },
  {
    key: 'category',
    label: 'Category',
    sortable: true
  },
  {
    key: 'thumbnail',
    label: 'Thumbnail'
  }];

const sort = ref({ column: 'title', direction: 'asc' as const })
const sortedRows = computed(() => {
  const sortedProducts = [...productslist]
  const { column, direction } = sort.value

  if (column && direction) {
    sortedProducts.sort((a, b) => {
      const aValue = a[column]
      const bValue = b[column]
      if (aValue < bValue) return direction === 'asc' ? -1 : 1
      if (aValue > bValue) return direction === 'asc' ? 1 : -1
      return 0
    })
  }

  return sortedProducts
});

const { data } = await useFetch<any>('https://dummyjson.com/products');
let productslist = data.value.products;

const q = ref('');
const page = ref(1);
const pageCount = 5;

const filteredData = computed(() => {
  if (!q.value) {
    return productslist;
  }

  return productslist.filter((product) => {
    return Object.values(product).some((value) => {
      return String(value).toLowerCase().includes(q.value.toLowerCase());
    });
  });
});



const rows = computed(() => {
  let filteredProducts = [...sortedRows.value]
  if (q.value) {
    filteredProducts = filteredProducts.filter(product => {
      return Object.values(product).some(value => {
        return String(value).toLowerCase().includes(q.value.toLowerCase())
      })
    })
  }
  const startIndex = (page.value - 1) * pageCount
  const endIndex = startIndex + pageCount
  return filteredProducts.slice(startIndex, endIndex)
});

watch(q,() => {
  page.value = 1;
});
</script>
<template>
  <title>Product List</title>
  <div class ="border-solid border-2 border-red-600 rounded-md ">
    <div class="  flex justify-center px-3 py-3.5 border-b border-gray-200 dark:border-gray-700">
      <UInput v-model="q" placeholder="Filter people..." />
    </div>
    <UTable :rows="rows" :columns="columns" v-model:sort="sort"
            sort-mode="manual">
      <template #thumbnail-data="{row}">
        <img :src="row.thumbnail" alt="" class="w-[100px] h-[100px]">
      </template>
      <template #description-data="{row}">
        <div class="text-wrap">{{ row.description }}</div>
      </template>
      <template #rating-data="{row}">
        <div :class="{ 'bg-red-500/20': row.rating < 4.5, 'bg-green-500/20': row.rating >= 4.5 }" class="px-3s flex justify-center py-2">
          {{ row.rating }}
        </div>
      </template>

    </UTable>
    <div class="flex justify-end px-3 py-3.5 border-t border-gray-200 dark:border-gray-700">
      <UPagination v-model="page" :page-count="pageCount" :total="productslist.length" />
    </div>
  </div>
</template>

<style>
</style>