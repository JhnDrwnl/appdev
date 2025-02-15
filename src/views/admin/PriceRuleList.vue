<!-- views/admin/PriceRuleList.vue -->
<template>
    <div class="w-full">
      <Alert 
        v-if="showAlert" 
        :type="alertType" 
        :dismissible="true" 
        @update:modelValue="showAlert = $event"
      >
        {{ alertMessage }}
      </Alert>
  
      <div v-if="currentView === 'list'">
        <div class="flex items-center justify-between mb-6">
          <div class="relative flex-1 max-w-md">
            <input
              type="text"
              placeholder="Search Price Rules"
              v-model="searchQuery"
              class="w-full px-4 py-2 pl-10 pr-4 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-[#FF9934]"
            />
            <span class="absolute left-3 top-2.5 text-gray-400">
              <MagnifyingGlassIcon class="w-5 h-5" />
            </span>
          </div>
          <button
            @click="showAddPriceRule"
            class="bg-[#FF9934] text-white px-4 py-2 rounded-full hover:bg-[#E88820] focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-[#FF9934]"
          >
            Add Price Rule
          </button>
        </div>
  
        <div class="bg-white rounded-lg shadow overflow-hidden">
          <div class="overflow-x-auto relative">
            <table class="w-full divide-y divide-gray-200">
              <thead class="bg-gray-50">
                <tr>
                  <th @click="sort('name')" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer">
                    Rule Name
                    <ArrowsUpDownIcon v-if="sortColumn === 'name'" :class="{ 'transform rotate-180': sortDirection === 'desc' }" class="inline-block w-4 h-4 ml-1" />
                  </th>
                  <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                    Type
                  </th>
                  <th @click="sort('value')" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider cursor-pointer">
                    Value
                    <ArrowsUpDownIcon v-if="sortColumn === 'value'" :class="{ 'transform rotate-180': sortDirection === 'desc' }" class="inline-block w-4 h-4 ml-1" />
                  </th>
                  <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                    Validity
                  </th>
                  <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">
                    Actions
                  </th>
                </tr>
              </thead>
              <tbody class="bg-white divide-y divide-gray-200">
                <tr v-for="rule in paginatedPriceRules" :key="rule.id">
                  <td class="px-6 py-4 whitespace-nowrap">
                    <div class="flex items-center">
                      <div class="text-sm font-medium text-gray-900">{{ rule.name }}</div>
                      <span :class="['ml-2 px-2 inline-flex text-xs leading-5 font-semibold rounded-full', getStatus(rule).class]">
                        {{ getStatus(rule).text }}
                      </span>
                    </div>
                  </td>
                  <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                    {{ rule.type === 'percentage' ? 'Percentage' : 'Fixed Amount' }}
                  </td>
                  <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                    {{ formatValue(rule) }}
                  </td>
                  <td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">
                    {{ formatValidity(rule) }}
                  </td>
                  <td class="px-6 py-4 whitespace-nowrap text-right text-sm font-medium">
                    <div class="flex items-center space-x-2">
                      <button
                        @click="editPriceRule(rule)"
                        class="text-green-600 hover:text-green-800 focus:outline-none"
                      >
                        <PencilSquareIcon class="h-5 w-5" />
                      </button>
                      <button
                        @click="confirmDeletePriceRule(rule.id)"
                        class="text-red-600 hover:text-red-800 focus:outline-none"
                      >
                        <TrashIcon class="h-5 w-5" />
                      </button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
  
        <div class="flex items-center justify-between mt-6">
          <div class="text-sm text-gray-500">
            Rows per page: 
            <select v-model="rowsPerPage" class="ml-2 border-gray-300 rounded-md">
              <option>5</option>
              <option>10</option>
              <option>25</option>
            </select>
          </div>
          <div class="flex items-center space-x-2">
            <button
              @click="previousPage"
              :disabled="currentPage === 1"
              class="p-2 rounded-md hover:bg-gray-100 disabled:opacity-50"
            >
              <ChevronLeftIcon class="h-5 w-5" />
            </button>
            <span class="text-sm text-gray-500">
              {{ currentPage }} of {{ totalPages }}
            </span>
            <button
              @click="nextPage"
              :disabled="currentPage === totalPages"
              class="p-2 rounded-md hover:bg-gray-100 disabled:opacity-50"
            >
              <ChevronRightIcon class="h-5 w-5" />
            </button>
          </div>
        </div>
      </div>
  
      <AddPriceRule
        v-else-if="currentView === 'add'"
        @cancel="showList"
        @save="handleSavePriceRule"
      />
  
      <EditPriceRule
        v-else-if="currentView === 'edit'"
        :priceRule="selectedPriceRule"
        @cancel="showList"
        @save="handleUpdatePriceRule"
      />
  
      <!-- Delete Confirmation Modal -->
      <div v-if="showDeleteModal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full z-50">
        <div class="relative top-20 mx-auto p-5 border w-96 shadow-lg rounded-md bg-white">
          <div class="mt-3 text-center">
            <h3 class="text-lg leading-6 font-medium text-gray-900">Confirm Deletion</h3>
            <div class="mt-2 px-7 py-3">
              <p class="text-sm text-gray-500">
                Are you sure you want to delete this price rule? This action cannot be undone.
              </p>
            </div>
            <div class="items-center px-4 py-3">
              <button
                @click="deletePriceRule"
                class="px-4 py-2 bg-red-600 text-white text-base font-medium rounded-full w-full shadow-sm hover:bg-red-700 focus:outline-none focus:ring-2 focus:ring-red-500 mb-2"
              >
                Delete
              </button>
              <button
                @click="cancelDelete"
                class="px-4 py-2 bg-gray-200 text-gray-800 text-base font-medium rounded-full w-full shadow-sm hover:bg-gray-300 focus:outline-none focus:ring-2 focus:ring-gray-400"
              >
                Cancel
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, computed, onMounted, watch } from 'vue'
  import { usePriceRuleStore } from '@/store/modules/priceRules'
  import AddPriceRule from '@/components/admin/AddPriceRule.vue'
  import EditPriceRule from '@/components/admin/EditPriceRule.vue'
  import Alert from '@/components/common/Alert.vue'
  import { 
    MagnifyingGlassIcon, 
    ArrowsUpDownIcon, 
    PencilSquareIcon, 
    TrashIcon,
    ChevronLeftIcon,
    ChevronRightIcon
  } from '@heroicons/vue/24/outline'
  
  const priceRuleStore = usePriceRuleStore()
  const currentView = ref('list')
  const searchQuery = ref('')
  const selectedPriceRule = ref(null)
  const showAlert = ref(false)
  const alertType = ref('')
  const alertMessage = ref('')
  const showDeleteModal = ref(false)
  const priceRuleToDelete = ref(null)
  
  const sortColumn = ref('name')
  const sortDirection = ref('asc')
  const currentPage = ref(1)
  const rowsPerPage = ref(10)
  
  onMounted(async () => {
    await priceRuleStore.fetchPriceRules()
  })
  
  const priceRules = computed(() => priceRuleStore.priceRules)
  
  const filteredPriceRules = computed(() => {
    return priceRules.value.filter(rule =>
      rule.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
  })
  
  const sortedPriceRules = computed(() => {
    return [...filteredPriceRules.value].sort((a, b) => {
      let modifier = sortDirection.value === 'asc' ? 1 : -1
      if (a[sortColumn.value] < b[sortColumn.value]) return -1 * modifier
      if (a[sortColumn.value] > b[sortColumn.value]) return 1 * modifier
      return 0
    })
  })
  
  const totalPages = computed(() => Math.ceil(sortedPriceRules.value.length / rowsPerPage.value))
  
  const paginatedPriceRules = computed(() => {
    const start = (currentPage.value - 1) * rowsPerPage.value
    const end = start + rowsPerPage.value
    return sortedPriceRules.value.slice(start, end)
  })
  
  const formatValue = (rule) => {
    if (rule.type === 'percentage') {
      return `${rule.value}%`
    } else if (rule.type === 'fixed') {
      return `₱${rule.value.toFixed(2)}`
    }
    return rule.value
  }
  
  const formatValidity = (rule) => {
    const now = new Date()
    const startDate = new Date(rule.startDate)
    const endDate = rule.endDate ? new Date(rule.endDate) : null
  
    const formatDateTime = (date) => {
      return date.toLocaleString('en-US', {
        year: 'numeric',
        month: 'short',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
        hour12: true
      })
    }
  
    if (startDate > now) {
      return `Starts on ${formatDateTime(startDate)}`
    } else if (endDate && endDate < now) {
      return `Ended on ${formatDateTime(endDate)}`
    } else if (endDate) {
      return `Valid until ${formatDateTime(endDate)}`
    } else {
      return 'Always valid'
    }
  }
  
  const getStatus = (rule) => {
    const now = new Date()
    const startDate = new Date(rule.startDate)
    const endDate = rule.endDate ? new Date(rule.endDate) : null
    const in24Hours = new Date(now.getTime() + 24 * 60 * 60 * 1000)
  
    if (startDate > now) {
      return { text: 'Scheduled', class: 'bg-blue-100 text-blue-800' }
    } else if (endDate && endDate < now) {
      return { text: 'Ended', class: 'bg-gray-100 text-gray-800' }
    } else if (endDate && endDate <= in24Hours) {
      return { text: 'Ending Soon', class: 'bg-yellow-100 text-yellow-800' }
    } else if (!endDate || endDate > now) {
      return { text: 'Active', class: 'bg-green-100 text-green-800' }
    }
  }
  
  const showAddPriceRule = () => {
    currentView.value = 'add'
    priceRuleStore.clearError()
  }
  
  const showList = () => {
    currentView.value = 'list'
    selectedPriceRule.value = null
    priceRuleStore.clearError()
  }
  
  const editPriceRule = (rule) => {
    selectedPriceRule.value = { ...rule }
    currentView.value = 'edit'
    priceRuleStore.clearError()
  }
  
  const confirmDeletePriceRule = (id) => {
    priceRuleToDelete.value = id
    showDeleteModal.value = true
  }
  
  const deletePriceRule = async () => {
    if (priceRuleToDelete.value) {
      const result = await priceRuleStore.deletePriceRule(priceRuleToDelete.value)
      if (result.success) {
        showAlertWithTimeout('success', 'Price rule deleted successfully')
      } else {
        showAlertWithTimeout('error', `Error deleting price rule: ${result.error}`)
      }
      showDeleteModal.value = false
      priceRuleToDelete.value = null
    }
  }
  
  const cancelDelete = () => {
    showDeleteModal.value = false
    priceRuleToDelete.value = null
  }
  
  const handleSavePriceRule = async (newRule) => {
    try {
      const result = await priceRuleStore.addPriceRule(newRule)
      if (result.success) {
        showAlertWithTimeout('success', 'Price rule added successfully')
        await priceRuleStore.fetchPriceRules() // Refresh the list
        showList()
      } else {
        throw new Error(result.error)
      }
    } catch (error) {
      showAlertWithTimeout('error', `Error adding price rule: ${error.message}`)
    }
  }
  
  const handleUpdatePriceRule = async (updatedRule) => {
    try {
      const result = await priceRuleStore.updatePriceRule(updatedRule.id, updatedRule)
      if (result.success) {
        showAlertWithTimeout('success', 'Price rule updated successfully')
        showList()
      } else {
        throw new Error(result.error)
      }
    } catch (error) {
      showAlertWithTimeout('error', `Error updating price rule: ${error.message}`)
    }
  }
  
  const sort = (column) => {
    if (sortColumn.value === column) {
      sortDirection.value = sortDirection.value === 'asc' ? 'desc' : 'asc'
    } else {
      sortColumn.value = column
      sortDirection.value = 'asc'
    }
  }
  
  const previousPage = () => {
    if (currentPage.value > 1) {
      currentPage.value--
    }
  }
  
  const nextPage = () => {
    if (currentPage.value < totalPages.value) {
      currentPage.value++
    }
  }
  
  watch(searchQuery, () => {
    currentPage.value = 1
  })
  
  watch(rowsPerPage, () => {
    currentPage.value = 1
  })
  
  const showAlertWithTimeout = (type, message) => {
    showAlert.value = true
    alertType.value = type
    alertMessage.value = message
    setTimeout(() => {
      showAlert.value = false
    }, 3000)
  }
  </script>
  
  <style scoped>
  .rounded-full {
    border-radius: 9999px;
  }
  
  .shadow {
    box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
  }
  
  .transition {
    transition-property: background-color, border-color, color, fill, stroke, opacity, box-shadow, transform;
    transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
    transition-duration: 150ms;
  }
  
  .ease-in-out {
    transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  }
  
  .duration-200 {
    transition-duration: 200ms;
  }
  </style>