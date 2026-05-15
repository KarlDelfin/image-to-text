<template>
  <div class="page">
    <div class="card">
      <div class="header">
        <h1>Image to Text</h1>
      </div>

      <!-- Upload Section -->
      <div class="upload_section">
        <label class="label">Upload Image</label>

        <div class="upload_group">
          <input
            class="file_input"
            type="file"
            @change="selectImage"
            ref="fileInput"
            accept=".jpeg, .jpg, .png"
          />

          <button class="clear_btn" :disabled="text == ''" @click="clearText">✕</button>
        </div>

        <small class="hint"> You can also paste an image using Ctrl + V </small>
      </div>

      <!-- Image Preview -->
      <div v-if="base64" class="preview">
        <img :src="base64" />
      </div>

      <!-- Extracted Text -->
      <div class="text_section">
        <div class="text_header">
          <label class="label">Extracted Text</label>

          <button class="copy_btn" @click="copyText" :disabled="text == ''">Copy</button>
        </div>

        <textarea
          v-model="text"
          rows="10"
          placeholder="Extracted text will appear here..."
        ></textarea>
      </div>
    </div>
  </div>
</template>

<script>
import { ElLoading, ElMessage } from 'element-plus'
import { createWorker } from 'tesseract.js'

export default {
  data() {
    return {
      base64: '',
      inputFile: [],
      text: '',
      file: [],
    }
  },

  methods: {
    copyText() {
      ElMessage.success('Text copied to clipboard')
      navigator.clipboard.writeText(this.text)
    },

    clearText() {
      this.text = ''
      const dataTransfer = new DataTransfer()
      this.$refs.fileInput.files = dataTransfer.files
      this.base64 = ''
    },

    pasteImage(e) {
      const file = e.clipboardData.files[0]
      this.file = file
      const fileType = file.type
      if (fileType === 'image/jpeg' || fileType === 'image/jpg' || fileType === 'image/png') {
        const dataTransfer = new DataTransfer()
        dataTransfer.items.add(file)
        this.$refs.fileInput.files = dataTransfer.files

        this.convertImageToText()

        const reader = new FileReader()
        reader.readAsDataURL(file)
        reader.onload = async () => {
          const base64 = reader.result
          this.base64 = base64
        }
        return
      }

      console.log('Please select an image only')
    },

    selectImage(e) {
      const file = e.target.files[0]
      this.file = file

      this.convertImageToText()

      const reader = new FileReader()
      reader.readAsDataURL(file)
      reader.onload = async () => {
        const base64 = reader.result
        this.base64 = base64
      }
    },

    async convertImageToText() {
      const loading = ElLoading.service({
        lock: true,
        text: 'Loading',
        background: 'rgba(0, 0, 0, 0.7)',
      })
      try {
        const worker = await createWorker('eng')
        const ret = await worker.recognize(this.file)
        this.text = ret.data.text
        await worker.terminate()
        loading.close()
      } catch {
        loading.close()
      }
    },
  },

  mounted() {
    window.addEventListener('paste', this.pasteImage)
  },
}
</script>
<style scoped>
.page {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #eef2ff, #f8fafc);
  padding: 40px 20px;
  font-family: Arial, Helvetica, sans-serif;
}

.card {
  width: 100%;
  max-width: 750px;
  background: #ffffff;
  padding: 30px;
  border-radius: 16px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.08);
}

.header {
  text-align: center;
  margin-bottom: 30px;
}

.header h1 {
  margin: 0;
  font-size: 28px;
  font-weight: 700;
  color: #1e293b;
}

.header p {
  margin-top: 8px;
  color: #64748b;
  font-size: 14px;
}

.label {
  font-weight: 600;
  font-size: 14px;
  color: #334155;
  margin-bottom: 6px;
  display: block;
}

.upload_section {
  margin-bottom: 25px;
}

.upload_group {
  display: flex;
  gap: 10px;
}

.file_input {
  flex: 1;
  padding: 10px;
  border-radius: 8px;
  border: 1px solid #cbd5e1;
  cursor: pointer;
  width: 100%;
}

.clear_btn {
  padding: 0 15px;
  border: none;
  border-radius: 8px;
  background: #ef4444;
  color: white;
  font-weight: bold;
  cursor: pointer;
  transition: 0.2s;
}

.clear_btn:disabled {
  background: #cbd5e1;
  cursor: not-allowed;
}

.clear_btn:hover:not(:disabled) {
  background: #dc2626;
}

.hint {
  font-size: 12px;
  color: #64748b;
  margin-top: 5px;
  display: block;
}

.preview {
  text-align: center;
  margin-bottom: 25px;
}

.preview img {
  max-height: 250px;
  max-width: 100%;
  border-radius: 12px;
  box-shadow: 0 10px 25px rgba(0, 0, 0, 0.08);
}

.text_section {
  margin-bottom: 20px;
}

.text_header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.copy_btn {
  padding: 6px 12px;
  font-size: 13px;
  border: none;
  border-radius: 6px;
  background: #3b82f6;
  color: white;
  cursor: pointer;
  transition: 0.2s;
}

.copy_btn:hover:not(:disabled) {
  background: #2563eb;
}

.copy_btn:disabled {
  background: #cbd5e1;
  cursor: not-allowed;
}

textarea {
  width: 100%;
  padding: 12px;
  border-radius: 10px;
  border: 1px solid #cbd5e1;
  resize: none;
  font-size: 14px;
  line-height: 1.5;
  background: #f8fafc;
}
</style>
