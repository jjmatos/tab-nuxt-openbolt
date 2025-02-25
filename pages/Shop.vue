<template>
<div class="space-y-6">
    <h2 class="text-3xl font-bold text-white">Product List</h2>
    <div class="bg-gray-800 rounded-xl shadow-xl p-6">
      <!-- CSV Upload -->
      <div class="mb-6">
        <label class="block text-sm font-medium text-gray-300 mb-2">
          Upload CSV File
        </label>
        <input
          type="file"
          accept=".csv"
          @change="handleFileUpload"
          class="block w-full text-sm text-gray-300
            file:mr-4 file:py-2 file:px-4
            file:rounded-full file:border-0
            file:text-sm file:font-semibold
            file:bg-indigo-600 file:text-white
            hover:file:bg-indigo-700
            cursor-pointer"
        />
      </div>

      <p class="text-gray-400 mb-4">
        Click on column headers to sort the table. Double-click a cell in a selected row to edit its value.
        Press Enter to save or Escape to cancel.
      </p>
      
      <DataTable 
        :headers="headers"
        :data="products"
        @selection-change="handleSelectionChange"
        @update:data="updateProducts"
      />

      <!-- Selected rows info -->
      <div v-if="selectedRows.length" class="mt-4 p-4 bg-gray-700 rounded-lg text-gray-200">
        <h3 class="font-semibold mb-2">Selected Products:</h3>
        <ul class="list-disc list-inside">
          <li v-for="index in selectedRows" :key="index">
            {{ formatSelectedRow(products[index]) }}
          </li>
        </ul>
      </div>
    </div>
</div>
</template>

<script setup>
import { ref } from 'vue'

const headers = ref(['name', 'price', 'stock']) // Default headers
const products = ref([
  { name: 'Laptop', price: '$999', stock: 5 },
  { name: 'Smartphone', price: '$699', stock: 10 },
  { name: 'Headphones', price: '$199', stock: 15 },
  { name: 'Mouse', price: '$49', stock: 20 }
])

const selectedRows = ref([])

const handleSelectionChange = (selected) => {
  selectedRows.value = selected
}

const updateProducts = (newProducts) => {
  products.value = newProducts
}

const formatSelectedRow = (row) => {
  return Object.entries(row)
    .map(([key, value]) => `${key}: ${value}`)
    .join(', ')
}

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  if (!file) return

  const reader = new FileReader()
  reader.onload = (e) => {
    const text = e.target.result
    const rows = text.split('\n')
    
    // Get and clean headers from first row
    const csvHeaders = rows[0]
      .split(';')
      .map(h => h.trim())
      .filter(h => h)
    
    if (csvHeaders.length === 0) {
      alert('CSV file must contain headers')
      return
    }

    // Update headers
    headers.value = csvHeaders

    // Parse data rows
    const newProducts = rows.slice(1)
      .filter(row => row.trim())
      .map(row => {
        const values = row.split(';').map(v => v.trim())
        const product = {}
        
        csvHeaders.forEach((header, index) => {
          let value = values[index] || ''
          
          if (!isNaN(value) && value !== '') {
            value = Number(value)
          }
          
          if (header.toLowerCase().includes('price') && typeof value === 'number') {
            value = `$${value}`
          } else if (typeof value === 'string' && value.startsWith('$')) {
            value = value
          }
          
          product[header] = value
        })
        
        return product
      })

    products.value = newProducts
  }
  reader.readAsText(file)
}
</script>
