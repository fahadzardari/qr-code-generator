<template>
  <div id="app">
    <header>
      <h1>QR Code Studio</h1>
      <p class="subtitle">Modern, customizable QR code generator</p>
    </header>

    <main class="main-layout">
      <!-- Settings Column -->
      <section class="card settings">
        <form @submit.prevent="generateQr">
          <div class="form-group">
            <label for="url">URL or Text</label>
            <input
              type="text"
              id="url"
              v-model="storedUrl"
              placeholder="https://example.com"
              required
            >
          </div>

          <div class="form-row" style="display: grid; grid-template-columns: 1fr 1fr; gap: 1rem;">
            <div class="form-group">
              <label for="fgColor">Foreground Color</label>
              <input type="color" id="fgColor" v-model="fgColor">
            </div>
            <div class="form-group">
              <label for="bgColor">Background Color</label>
              <input type="color" id="bgColor" v-model="bgColor">
            </div>
          </div>

          <div class="form-row" style="display: grid; grid-template-columns: 1fr 1fr; gap: 1rem;">
            <div class="form-group">
              <label for="size">Size (px)</label>
              <input type="number" id="size" v-model.number="size" min="100" max="1000">
            </div>
            <div class="form-group">
              <label for="level">Correction Level</label>
              <select id="level" v-model="level">
                <option value="L">Low (7%)</option>
                <option value="M">Medium (15%)</option>
                <option value="Q">Quartile (25%)</option>
                <option value="H">High (30%)</option>
              </select>
            </div>
          </div>

          <button type="submit" class="primary">
            <span>Generate QR Code</span>
          </button>
        </form>
      </section>

      <!-- Preview Column -->
      <section class="card preview">
        <div v-if="generated" class="preview-container">
          <div class="qr-wrapper" id="qr-container">
            <qr
              id="qrCode"
              :value="url"
              :size="size"
              :level="level"
              :foreground="fgColor"
              :background="bgColor"
            />
          </div>
          <div class="button-group">
            <button @click="downloadQr" class="secondary">
              <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><path d="M21 15v4a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2v4"/><polyline points="7 10 12 15 17 10"/><line x1="12" y1="15" x2="12" y2="3"/></svg>
              Download
            </button>
            <button @click="copyToClipboard" class="secondary">
              <svg xmlns="http://www.w3.org/2000/svg" width="20" height="20" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"><rect x="9" y="9" width="13" height="13" rx="2" ry="2"/><path d="M5 15H4a2 2 0 0 1-2-2V4a2 2 0 0 1 2-2h9a2 2 0 0 1 2 2v1"/></svg>
              Copy
            </button>
          </div>
        </div>
        <div v-else class="preview-container">
          <p class="empty-state">Enter text and click generate to preview</p>
        </div>
      </section>
    </main>

    <!-- History Section -->
    <section class="history-section">
      <h3>Recent Generations</h3>
      <div v-if="history.length > 0" class="history-list">
        <div
          v-for="(item, index) in history"
          :key="index"
          class="history-item"
          @click="loadFromHistory(item)"
          :title="item.url"
        >
          {{ item.url }}
        </div>
      </div>
      <p v-else class="empty-state">No recent history</p>
    </section>
  </div>
</template>

<script>
import qr from 'qrcode.vue';

export default {
  components: {
    qr,
  },
  data() {
    return {
      url: '',
      storedUrl: '',
      generated: false,
      size: 200,
      level: 'H',
      fgColor: '#000000',
      bgColor: '#ffffff',
      history: []
    }
  },
  mounted() {
    const savedHistory = localStorage.getItem('qr_history');
    if (savedHistory) {
      this.history = JSON.parse(savedHistory);
    }
  },
  methods: {
    generateQr() {
      if (this.storedUrl.length > 0) {
        this.url = this.storedUrl;
        this.generated = true;
        this.addToHistory();
      }
    },
    downloadQr() {
      const canvas = document.querySelector("#qrCode");
      if (!canvas) return;

      const fileName = this.url.replace(/[^a-z0-9]/gi, '_').substring(0, 30) || 'qr-code';
      const link = document.createElement('a');
      link.download = `${fileName}.png`;
      link.href = canvas.toDataURL();
      link.click();
    },
    copyToClipboard() {
      const canvas = document.querySelector("#qrCode");
      if (!canvas) return;

      canvas.toBlob((blob) => {
        const item = new ClipboardItem({ "image/png": blob });
        navigator.clipboard.write([item]).then(() => {
          alert("QR Code copied to clipboard!");
        }).catch(err => {
          console.error("Could not copy image: ", err);
        });
      });
    },
    addToHistory() {
      const newItem = {
        url: this.url,
        size: this.size,
        level: this.level,
        fgColor: this.fgColor,
        bgColor: this.bgColor
      };

      // Remove duplicate if exists
      this.history = this.history.filter(item => item.url !== this.url);

      // Add to front
      this.history.unshift(newItem);

      // Keep only last 5
      if (this.history.length > 5) {
        this.history.pop();
      }

      localStorage.setItem('qr_history', JSON.stringify(this.history));
    },
    loadFromHistory(item) {
      this.storedUrl = item.url;
      this.url = item.url;
      this.size = item.size;
      this.level = item.level;
      this.fgColor = item.fgColor;
      this.bgColor = item.bgColor;
      this.generated = true;
    }
  }
}
</script>
