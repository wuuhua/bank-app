<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import { useRoute, useRouter } from 'vue-router'

const bankData = ref([])
const searchBank = ref('')
const isDropdownOpen = ref(false)
const selectedBank = ref(null)
const searchBranch = ref('')
const isBranchDropdownOpen = ref(false)
const selectedBranch = ref(null)
const sectionKey = ref('')
const highlightedIndex = ref(-1)
const branchHighlightedIndex = ref(-1)
const inputRef = ref(null)
const dropdownRef = ref(null)
const branchInputRef = ref(null)
const branchDropdownRef = ref(null)
const didSelectBank = ref(false)
const didSelectBranch = ref(false)
const oldSearchBank = ref('')
const oldSearchBranch = ref('')

const route = useRoute()
const router = useRouter()

const loadBankData = async () => {
  try {
    const response = await fetch('/twbank.json')
    const data = await response.json()
    bankData.value = Object.values(
      data.reduce((result, item) => {
        const bankMatch = item.機構名稱.match(/^(.+?(銀行|辦事處|信用合作社))/) 
        const bankName = bankMatch ? bankMatch[1] : "未知機構"
        const branchName = item.機構名稱.replace(bankName, "").trim() 
        const bankKey = `${item.總機構代號} ${bankName}`
        
        if (bankName === "未知機構") {
          return result
        }

        if (!result[bankKey]) {
          result[bankKey] = {
            bankCode: String(item.總機構代號),
            bankName: bankName,
            branches: [],
          }
        }
  
        result[bankKey].branches.push({
          branchName: branchName,
          branchCode: String(item.機構代號),
          address: item.地址,
          telephone: item.電話,
        })
  
        return result
      }, {})      
    )

    if (route.params.bankCode && route.params.branchCode && route.params.branchName) {
      const bank = bankData.value.find(b => b.bankCode === route.params.bankCode)
      if (bank) {
        searchBank.value = `${bank.bankCode} ${bank.bankName}`
        selectedBank.value = bank
        const branch = bank.branches.find(br => br.branchCode === route.params.branchCode)
        if (branch) {
          searchBranch.value = branch.branchName
          selectedBranch.value = branch
        }
      }
    }
  } catch (error) {
    console.error(error)
    alert('無法取得銀行資料')
  }
}

const filteredBanks = computed(() => {
  
  if (!searchBank.value) return bankData.value || []
  
  const keyword = searchBank.value.trim()
  const filtered = bankData.value.filter(bank => 
    bank.bankName.includes(keyword) ||
    String(bank.bankCode).includes(keyword)
  )
  return filtered
})

const filteredBranches = computed(() => {
  if (!selectedBank.value) return []
  if (!searchBranch.value) return selectedBank.value.branches || []
  const keyword = searchBranch.value.trim()
  const filtered = selectedBank.value.branches.filter(branch => 
    branch.branchName.includes(keyword)
  )
  return filtered
})

const handleInputClick = () => {
  didSelectBank.value = false
  oldSearchBank.value = searchBank.value
  isDropdownOpen.value = true
  if (selectedBank.value) return searchBank.value = null
}

const handleBranchInputClick = () => {
  didSelectBranch.value = false
  oldSearchBranch.value = searchBranch.value
  if (selectedBank.value) {
    isBranchDropdownOpen.value = true
    searchBranch.value = '' 
  }
}

const handleBankSelect = (bank) => {
  didSelectBank.value = true
  selectedBank.value = bank
  selectedBranch.value = null
  searchBank.value = `${bank.bankCode} ${bank.bankName}`
  searchBranch.value = null
  isDropdownOpen.value = false
}

const handleBranchSelect = (branch) => {
  didSelectBranch.value = true
  selectedBranch.value = branch
  searchBranch.value = branch.branchName
  isBranchDropdownOpen.value = false
  const branchDisplayName = `${selectedBank.value.bankName}-${branch.branchName}`
  router.push({ name: 'BankBranch', params: { bankCode: selectedBank.value.bankCode, branchCode: branch.branchCode, branchName: branchDisplayName } })

  sectionKey.value = `${selectedBank.value.bankCode}-${selectedBranch.value.branchCode}`
}

const handleKeyDown = (e) => {
  if (!isDropdownOpen.value) {
    if (e.key === 'ArrowDown' || e.key === 'Enter') {
      isDropdownOpen.value = true
      return
    }
  }

  switch (e.key) {
    case 'ArrowDown':
      highlightedIndex.value = Math.min(highlightedIndex.value + 1, filteredBanks.value.length - 1)
      break
    case 'ArrowUp':
      highlightedIndex.value = Math.max(highlightedIndex.value - 1, 0)
      break
    case 'Enter':
      if (highlightedIndex.value >= 0) {
        handleBankSelect(filteredBanks.value[highlightedIndex.value])
      }
      break
      case 'Escape':
      isDropdownOpen.value = false
      break
    default:
      break
  }
}

const handleBranchKeyDown = (e) => {
  if (!isBranchDropdownOpen.value) {
    if (e.key === 'ArrowDown' || e.key === 'Enter') {
      isBranchDropdownOpen.value = true
      return
    }
  }

  switch (e.key) {
    case 'ArrowDown':
      branchHighlightedIndex.value = Math.min(branchHighlightedIndex.value + 1, filteredBranches.value.length - 1)
      break
    case 'ArrowUp':
      branchHighlightedIndex.value = Math.max(branchHighlightedIndex.value - 1, 0)
      break
    case 'Enter':
      if (branchHighlightedIndex.value >= 0) {
        handleBranchSelect(filteredBranches.value[branchHighlightedIndex.value])
      }
      break
    case 'Escape':
      isBranchDropdownOpen.value = false
      break
    default:
      break
  }
}

const handleClickOutside = (e) => {
  if (dropdownRef.value && !dropdownRef.value.contains(e.target) && inputRef.value && !inputRef.value.contains(e.target)) {
    isDropdownOpen.value = false
    if (!didSelectBank.value) {
      searchBank.value = oldSearchBank.value
    }
  }
  if (branchDropdownRef.value && !branchDropdownRef.value.contains(e.target) && branchInputRef.value && !branchInputRef.value.contains(e.target)) {
    isBranchDropdownOpen.value = false
    if (!didSelectBranch.value) {
      searchBranch.value = oldSearchBranch.value
    }
  }
}

const copyPageLink = () => {
  navigator.clipboard.writeText(window.location.href)
    .then(() => {
      alert('頁面連結已複製到剪貼簿')
    })
    .catch(err => {
      console.error('複製失敗:', err)
      alert('無法複製頁面連結')
    })
}

const copyBranchCode = () => {
  navigator.clipboard.writeText(selectedBranch.value.branchCode)
    .then(() => {
      alert('分行代碼已複製到剪貼簿')
    })
    .catch(err => {
      console.error('複製失敗:', err)
      alert('無法複製分行代碼')
    })
}

onMounted(() => {
  loadBankData()
  document.addEventListener('click', handleClickOutside)
})

onUnmounted(() => {
  document.removeEventListener('click', handleClickOutside)
})
</script>

<template>
  <main class="container mx-auto my-2 min-w-28">
    <section class="mx-2 sm:mx-0 px-2">
      <h1 class="text-4xl sm:text-5xl font-thin my-2 -ml-1">台灣銀行代碼查詢</h1>
      <div class="flex flex-col sm:flex-row gap-2 mt-2">
        <div class="flex flex-col relative">
          <label for="bank-name" class="font-bold text-gray-700">銀行名稱</label>
          <input
            ref="inputRef"
            id="bank-name"
            type="text"
            placeholder="請輸入關鍵字或銀行代碼"
            class="w-full p-2 border border-zinc-400 rounded font-normal text-ellipsis focus:outline-none focus:ring-1 focus:ring-blue-500"
            @click="handleInputClick"
            @keydown="handleKeyDown"
            v-model="searchBank"
          />
          <i class="fa-solid fa-angle-down absolute right-3 top-9 text-zinc-300 hover:text-zinc-400"></i>
          <p class="text-sm font-light text-zinc-400 p-1">使用下拉選單或直接輸入關鍵字查詢</p>

          <ul
            v-show="isDropdownOpen"
            ref="dropdownRef"
            class="absolute top-3/4 left-0 right-0 bg-white border border-zinc-400 rounded divide-y divide-dotted divide-zinc-400 shadow-lg max-h-64 overflow-y-auto z-10"
          >
            <li
              v-for="(bank, index) in filteredBanks"
              v-if="filteredBanks.length > 0"
              :key="searchBank"
              @click="handleBankSelect(bank)"
              class="cursor-pointer hover:bg-blue-100 px-2 py-1 h-fit"
              :class="{ 'bg-blue-100': index === highlightedIndex }"
            >
              {{ bank?.bankCode }} {{ bank?.bankName }}
            </li>
            <li v-else class="px-2 py-1 text-gray-500">
              沒有找到符合的銀行
            </li>
          </ul>
        </div>

        <div class="flex flex-col relative">
          <label for="branch-name" class="font-bold text-gray-700">分行名稱</label>
          <input
            ref="branchInputRef"
            id="branch-name"
            type="text"
            placeholder="請選擇分行名稱"
            class="w-full p-2 border border-zinc-400 rounded font-normal text-ellipsis focus:outline-none focus:ring-1 focus:ring-blue-500 disabled:bg-zinc-100"
            @click="handleBranchInputClick"
            @keydown="handleBranchKeyDown"
            v-model="searchBranch"
            :disabled="!selectedBank"
          />
          <i class="fa-solid fa-angle-down absolute right-3 top-9 text-zinc-300"></i>

          <ul
            v-show="isBranchDropdownOpen"
            ref="branchDropdownRef"
            class="absolute top-[70.2px] sm:top-3/4 left-0 right-0 bg-white border border-zinc-400 rounded divide-y divide-dotted divide-zinc-400 shadow-lg max-h-64 overflow-y-auto z-10"
          >
            <li
              v-for="(branch, index) in filteredBranches"
              :key="searchBranch"
              class="cursor-pointer hover:bg-blue-100 px-2 py-1"
              :class="{ 'bg-blue-100': index === branchHighlightedIndex }"
              @click="handleBranchSelect(branch)"
            >
              {{ branch?.branchName }}
            </li>
            <li v-if="isBranchDropdownOpen && filteredBranches.length === 0" class="px-2 py-1 text-gray-500">
              沒有找到符合的分行
            </li>
          </ul>
        </div>
      </div>
    </section>

    <section v-if="selectedBank && selectedBranch" :key="sectionKey" class="mt-4 sm:mt-2 min-w-fit">
      <div class="flex flex-col sm:flex-row justify-between items-start sm:items-end px-2 py-2 bg-green-50 rounded border border-dotted border-gray-700">
        <div>
          <h2 class="text-3xl mt-1 mb-2">{{ selectedBank?.bankName }}({{ selectedBank?.bankCode }}){{ selectedBranch?.branchName }}</h2>
          <p class="text-xl my-1">分行代碼：{{ selectedBranch?.branchCode }}
            <button @click="copyBranchCode" class="ml-2 hover:bg-green-400 bg-green-500 text-green-50 rounded border border-gray-600 px-2 py-1 text-sm">複製代碼</button>
          </p>
          <p class="text-xl my-1">地址：{{ selectedBranch?.address }}</p>
          <p class="text-xl my-1">電話：{{ selectedBranch?.telephone }}</p>
        </div>
        <footer class="text-sm text-green-900 text-right">
          資料來源：
          <a href="https://data.gov.tw/dataset/6041">政府資料開放平台</a>
        </footer>
      </div>
      <footer class="mt-2">
        <a href="/" class="mr-1 rounded border border-gray-600 px-2 py-1 text-sm">重新查詢</a>
        <button @click="copyPageLink" class="hover:bg-blue-400 bg-blue-500 text-blue-50 rounded border border-gray-600 px-2 py-1 text-sm">
          複製此頁面連結
        </button>
      </footer>
    </section>
  </main>
</template>