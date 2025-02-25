<template>
  <div>
    <!-- Feature Toggles -->
    <div class="mb-4 flex space-x-4">
      <div>
        <label class="inline-flex items-center">
          <input
            type="checkbox"
            v-model="enablePagination"
            class="mr-2 rounded border-gray-600 text-indigo-600 focus:ring-indigo-500"
          />
          <span class="text-gray-300">Enable Pagination</span>
        </label>
      </div>
      <div>
        <label class="inline-flex items-center">
          <input
            type="checkbox"
            v-model="enableAllColumnSearch"
            class="mr-2 rounded border-gray-600 text-indigo-600 focus:ring-indigo-500"
          />
          <span class="text-gray-300">Search All Columns</span>
        </label>
      </div>
    </div>

    <!-- Search input -->
    <div class="mb-4">
      <input
        type="text"
        v-model="searchQuery"
        placeholder="Search..."
        class="w-full px-4 py-2 bg-gray-700 border border-gray-600 rounded-lg 
               text-white placeholder-gray-400 focus:outline-none focus:ring-2 
               focus:ring-indigo-500 focus:border-transparent"
        :disabled="!enableAllColumnSearch"
      />
    </div>

    <!-- Copy First Result Button -->
    <div class="mb-4">
      <button
        @click="copyFirstResult"
        class="px-4 py-2 bg-indigo-600 text-white rounded-md"
        :disabled="filteredAndSortedData.length === 0"
      >
        Copy First Result (Ctrl+C)
      </button>
    </div>

    <div class="overflow-x-auto rounded-lg border border-gray-700">
      <table class="w-full border-collapse">
        <thead>
          <tr>
            <th class="bg-gray-700 px-4 py-3 border-b border-gray-600">
              <input
                type="checkbox"
                :checked="allSelected"
                @change="toggleAllRows"
                class="w-4 h-4 rounded border-gray-600 text-indigo-600 
                       focus:ring-indigo-500 focus:ring-offset-gray-700"
              />
            </th>
            <th v-for="header in headers" 
                :key="header"
                @click="sort(header)"
                class="bg-gray-700 px-4 py-3 border-b border-gray-600 
                       text-left cursor-pointer hover:bg-gray-600 transition-colors">
              <div class="flex items-center justify-between">
                <span class="font-semibold">{{ header }}</span>
                <span v-if="sortColumn === header" class="ml-2">
                  {{ sortDirection === 'asc' ? '↑' : '↓' }}
                </span>
              </div>
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(row, rowIndex) in paginatedData" 
              :key="rowIndex"
              class="hover:bg-gray-700/50 transition-colors"
              :class="{ 'bg-indigo-900/30': selectedRows.includes(rowIndex + startIndex) }">
            <td class="px-4 py-3 border-b border-gray-700">
              <input
                type="checkbox"
                :checked="selectedRows.includes(rowIndex + startIndex)"
                @change="toggleRow(rowIndex + startIndex)"
                class="w-4 h-4 rounded border-gray-600 text-indigo-600 
                       focus:ring-indigo-500 focus:ring-offset-gray-700"
              />
            </td>
            <td v-for="header in headers" 
                :key="header"
                class="px-4 py-3 border-b border-gray-700"
                @dblclick="startEditing(rowIndex + startIndex, header)">
              <div v-if="isEditing(rowIndex + startIndex, header)" class="flex">
                <input
                  ref="editInput"
                  :value="editValue"
                  @input="updateEditValue"
                  class="w-full px-2 py-1 bg-gray-600 border border-gray-500 rounded 
                         text-white focus:outline-none focus:ring-2 focus:ring-indigo-500"
                  @keyup.enter="saveEdit"
                  @keyup.esc="cancelEdit"
                  @blur="cancelEdit"
                  v-focus
                />
              </div>
              <div v-else>
                {{ row[header] }}
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </div>

    <!-- Pagination -->
    <div v-if="enablePagination" class="flex justify-center mt-4">
      <button
        @click="prevPage"
        :disabled="currentPage === 1"
        class="px-4 py-2 bg-gray-700 text-white rounded-md mr-2"
      >
        Previous
      </button>
      <button
        @click="nextPage"
        :disabled="currentPage * itemsPerPage >= filteredAndSortedData.length"
        class="px-4 py-2 bg-gray-700 text-white rounded-md"
      >
        Next
      </button>
    </div>

    <div class="mt-4 text-sm text-gray-400">
      {{ selectedRows.length }} row(s) selected
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'

const props = defineProps({
  headers: {
    type: Array,
    required: true
  },
  data: {
    type: Array,
    required: true
  }
})

const emit = defineEmits(['selection-change', 'update:data'])

// Feature toggles
const enablePagination = ref(true)
const enableAllColumnSearch = ref(false)

// Pagination
const currentPage = ref(1)
const itemsPerPage = 10

const startIndex = computed(() => {
  return (currentPage.value - 1) * itemsPerPage
})

const endIndex = computed(() => {
  return currentPage.value * itemsPerPage
})

// Editing state
const editingCell = ref(null)
const editValue = ref('')

// Focus directive
const vFocus = {
  mounted: (el) => el.focus()
}

const isEditing = (rowIndex, header) => {
  return editingCell.value && 
         editingCell.value.rowIndex === rowIndex && 
         editingCell.value.header === header
}

const startEditing = (rowIndex, header) => {
  if (selectedRows.value.includes(rowIndex)) {
    editingCell.value = { rowIndex, header }
    editValue.value = props.data[rowIndex][header]
  }
}

const updateEditValue = (event) => {
  editValue.value = event.target.value
}

const saveEdit = () => {
  if (!editingCell.value) return

  const { rowIndex, header } = editingCell.value
  const newData = [...props.data]
  newData[rowIndex] = { ...newData[rowIndex], [header]: editValue.value }
  emit('update:data', newData)
  editingCell.value = null
}

const cancelEdit = () => {
  editingCell.value = null
}

const sortColumn = ref('')
const sortDirection = ref('asc')
const searchQuery = ref('')
const selectedRows = ref([])

const sort = (header) => {
  if (sortColumn.value === header) {
    sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
  } else {
    sortColumn.value = header
    sortDirection.value = 'asc'
  }
}

const toggleRow = (index) => {
  const selectedIndex = selectedRows.value.indexOf(index)
  if (selectedIndex === -1) {
    selectedRows.value.push(index)
  } else {
    selectedRows.value.splice(selectedIndex, 1)
  }
  emit('selection-change', selectedRows.value)
}

const toggleAllRows = () => {
  if (selectedRows.value.length === filteredAndSortedData.value.length) {
    selectedRows.value = []
  } else {
    selectedRows.value = [...Array(filteredAndSortedData.value.length).keys()]
  }
  emit('selection-change', selectedRows.value)
}

const allSelected = computed(() => {
  return selectedRows.value.length === filteredAndSortedData.value.length
})

const filteredData = computed(() => {
  if (!searchQuery.value) return props.data

  const query = searchQuery.value.toLowerCase()
  return props.data.filter(row => {
    if (enableAllColumnSearch.value) {
      return Object.values(row).some(value => 
        String(value).toLowerCase().includes(query)
      )
    } else {
      return String(row[props.headers[1]]).toLowerCase().includes(query)
    }
  })
})

const filteredAndSortedData = computed(() => {
  if (!sortColumn.value) return filteredData.value

  return [...filteredData.value].sort((a, b) => {
    const aVal = a[sortColumn.value]
    const bVal = b[sortColumn.value]

    if (typeof aVal === 'number' && typeof bVal === 'number') {
      return sortDirection.value === 'asc' ? aVal - bVal : bVal - aVal
    }

    if (typeof aVal === 'string' && aVal.startsWith('$') &&
        typeof bVal === 'string' && bVal.startsWith('$')) {
      const aNum = parseFloat(aVal.replace('$', ''))
      const bNum = parseFloat(bVal.replace('$', ''))
      return sortDirection.value === 'asc' ? aNum - bNum : bNum - aNum
    }

    return sortDirection.value === 'asc' 
      ? String(aVal).localeCompare(String(bVal))
      : String(bVal).localeCompare(String(aVal))
  })
})

const paginatedData = computed(() => {
  if (enablePagination.value) {
    return filteredAndSortedData.value.slice(startIndex.value, endIndex.value)
  } else {
    return filteredAndSortedData.value
  }
})

const prevPage = () => {
  currentPage.value--
}

const nextPage = () => {
  currentPage.value++
}

const copyFirstResult = () => {
  if (filteredAndSortedData.value.length > 0) {
    const firstResult = filteredAndSortedData.value[0][props.headers[0]]
    navigator.clipboard.writeText(firstResult)
      .then(() => {
        alert('Copied to clipboard: ' + firstResult)
      })
      .catch(err => {
        console.error('Failed to copy text: ', err)
      })
  }
}

onMounted(() => {
  window.addEventListener('keydown', (event) => {
    if ((event.ctrlKey || event.metaKey) && event.key === 'c') {
      copyFirstResult()
    }
  })
})
</script>
