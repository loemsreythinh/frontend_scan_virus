<script setup>
import { ref } from 'vue'

const activeTab = ref('FILE') // 'FILE' | 'URL' | 'SEARCH'
const selectedFile = ref(null)
const inputUrl = ref('')
const searchKeyword = ref('')
const fileInputRef = ref(null)
const isLoading = ref(false)
const scanResult = ref(null)
const errorMessage = ref('')

// Base URL ទៅកាន់ Render Backend
const API_BASE_URL = 'https://backend-scanvirus-2.onrender.com'

const handleFileSelect = (event) => {
  const file = event.target.files[0]
  if (file) {
    selectedFile.value = file
    errorMessage.value = ''
    scanResult.value = null
  }
}

const handleDrop = (event) => {
  const file = event.dataTransfer.files[0]
  if (file) {
    selectedFile.value = file
    errorMessage.value = ''
    scanResult.value = null
  }
}

const clearSelection = () => {
  selectedFile.value = null
  inputUrl.value = ''
  searchKeyword.value = ''
  if (fileInputRef.value) fileInputRef.value.value = ''
  scanResult.value = null
  errorMessage.value = ''
}

const switchTab = (tab) => {
  activeTab.value = tab
  clearSelection()
}

const executeScan = async () => {
  errorMessage.value = ''
  scanResult.value = null
  isLoading.value = true

  try {
    let response

    if (activeTab.value === 'FILE') {
      if (!selectedFile.value) {
        errorMessage.value = 'Please select a file to scan.'
        isLoading.value = false
        return
      }
      const formData = new FormData()
      formData.append('file', selectedFile.value)
      
      // បាញ់ត្រង់ទៅ Render Backend
      response = await fetch(`${API_BASE_URL}/api/v1/scan/file`, { 
        method: 'POST', 
        body: formData 
      })
    } else if (activeTab.value === 'URL') {
      if (!inputUrl.value.trim()) {
        errorMessage.value = 'Please enter a valid URL.'
        isLoading.value = false
        return
      }
      response = await fetch(`${API_BASE_URL}/api/v1/scan/url`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ url: inputUrl.value.trim() })
      })
    } else {
      if (!searchKeyword.value.trim()) {
        errorMessage.value = 'Please enter a domain, IP, or file hash to search.'
        isLoading.value = false
        return
      }
      response = await fetch(`${API_BASE_URL}/api/v1/scan/url`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ url: searchKeyword.value.trim() })
      })
    }

    if (!response.ok) throw new Error(`Status: ${response.status}`)
    scanResult.value = await response.json()
  } catch (err) {
    console.error('API Error:', err)
    errorMessage.value = 'Scan failed. Check your backend connection.'
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="vt-page">
    <main class="vt-container">
      <div class="brand-section">
        <h1 class="vt-title">VIRUSTOTAL</h1>
        <p class="vt-subtitle">
          Analyse suspicious files, domains, IPs and URLs to detect malware and other breaches, automatically share them with the security community.
        </p>
      </div>

      <div class="tabs-header">
        <button 
          :class="['tab-item', { active: activeTab === 'FILE' }]" 
          @click="switchTab('FILE')"
        >
          FILE
        </button>
        <button 
          :class="['tab-item', { active: activeTab === 'URL' }]" 
          @click="switchTab('URL')"
        >
          URL
        </button>
        <button 
          :class="['tab-item', { active: activeTab === 'SEARCH' }]" 
          @click="switchTab('SEARCH')"
        >
          SEARCH
        </button>
      </div>

      <div v-if="activeTab === 'FILE'" class="tab-content" @dragover.prevent @drop.prevent="handleDrop">
        <input 
          id="vt-file" 
          ref="fileInputRef" 
          type="file" 
          class="hidden-input" 
          @change="handleFileSelect" 
        />

        <div class="vt-input-box">
          <label for="vt-file" class="btn-choose-file">
            {{ selectedFile ? selectedFile.name : 'Choose file' }}
          </label>
          <button class="btn-submit-action" :disabled="isLoading" @click="executeScan">
            {{ isLoading ? 'Scanning...' : 'Scan' }}
          </button>
          <button class="btn-cancel" :disabled="isLoading" @click="clearSelection">Cancel</button>
        </div>
      </div>

      <div v-else-if="activeTab === 'URL'" class="tab-content">
        <div class="vt-input-box">
          <input 
            v-model="inputUrl" 
            type="text" 
            placeholder="http://www.example.com" 
            @keyup.enter="executeScan" 
          />
          <button class="btn-submit-action" :disabled="isLoading" @click="executeScan">
            Scan
          </button>
          <button class="btn-cancel" :disabled="isLoading" @click="clearSelection">Cancel</button>
        </div>
      </div>

      <div v-else class="tab-content">
        <div class="vt-input-box">
          <input 
            v-model="searchKeyword" 
            type="text" 
            placeholder="URL, IP address, domain, or file hash" 
            @keyup.enter="executeScan" 
          />
          <button class="btn-submit-action" :disabled="isLoading" @click="executeScan">
            Search
          </button>
          <button class="btn-cancel" :disabled="isLoading" @click="clearSelection">Cancel</button>
        </div>
      </div>

      <p class="vt-terms">
        By submitting data above, you are agreeing to our <span>Terms of Service</span> and <span>Privacy Notice</span>, and to the <strong>sharing of your Sample submission with the security community</strong>. Please do not submit any personal information; we are not responsible for the contents of your submission. <span>Learn more</span>.
      </p>

      <div v-if="errorMessage" class="error-msg">
        {{ errorMessage }}
      </div>

      <div v-if="scanResult" class="result-card" :class="scanResult.status.toLowerCase()">
        <div class="result-head">
          <span class="status-badge" :class="scanResult.status.toLowerCase()">{{ scanResult.status }}</span>
          <span class="threat-score">Detections: <strong>{{ scanResult.threatScore }}/100</strong></span>
        </div>
        <p class="res-target"><strong>Target:</strong> {{ scanResult.target }}</p>
        <div class="detections-box">
          <li v-for="(item, idx) in scanResult.detections" :key="idx">{{ item }}</li>
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped>

</style>