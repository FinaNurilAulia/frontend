<template>
  <!-- Container utama dengan latar gelap dan teks terang -->
  <div class="container-fluid py-5" style="background-color: #0d0d0d; min-height: 100vh; color: #eee;">
    <div class="container">
      <!-- Header dan tombol dark mode -->
      <div class="d-flex justify-content-between align-items-center mb-4">
        <h2 class="fw-bold" style="color: #f14ff1;">🎫 Tiket Konser Terbaru</h2>
        <!-- Tombol dark mode (hanya menampilkan alert) -->
        <button @click="toggleDark" class="btn btn-outline-light">🌙</button>
      </div>

      <!-- Bagian pencarian dan sortir -->
      <div class="row mb-4">
        <!-- Input pencarian berdasarkan judul tiket -->
        <div class="col-md-6 mb-2">
          <input 
            v-model="search" 
            type="text" 
            class="form-control bg-dark text-light border border-pink"
            placeholder="🔍 Cari judul tiket..."
          />
        </div>
        <!-- Dropdown untuk sortir berdasarkan tanggal atau harga -->
        <div class="col-md-6 mb-2">
          <select 
            v-model="sortBy" 
            class="form-select bg-dark text-light border border-pink"
          >
            <option value="">Urutkan berdasarkan</option>
            <option value="date">📅 Tanggal</option>
            <option value="price">💰 Harga</option>
          </select>
        </div>
      </div>

      <!-- Tampilkan spinner jika data sedang dimuat -->
      <div v-if="loading" class="text-center">
        <div class="spinner-border text-light" role="status"></div>
        <p class="mt-2 text-muted">Memuat tiket konser...</p>
      </div>

      <!-- Tampilkan pesan error jika fetch gagal -->
      <div v-if="error" class="alert alert-danger text-center">
        Gagal memuat tiket. Silakan coba lagi nanti.
      </div>

      <!-- Tampilkan daftar tiket jika ada data dan tidak loading -->
      <div v-if="!loading && filteredTickets.length > 0" class="row">
        <!-- Loop setiap tiket dan tampilkan dalam card -->
        <div class="col-md-4 mb-4" v-for="ticket in filteredTickets" :key="ticket.id">
          <div class="card bg-gradient border-0 text-light shadow"
               style="background: linear-gradient(145deg, #2b2b2b, #1a1a1a);">
            <!-- Tampilkan gambar jika tersedia -->
            <img 
              v-if="ticket.image_url" 
              :src="ticket.image_url"
              class="card-img-top"
              style="height: 200px; object-fit: cover; border-bottom: 2px solid #f14ff1;" 
            />
            <div class="card-body">
              <!-- Tampilkan judul tiket -->
              <h5 class="card-title" style="color: #f14ff1;">🎵 {{ ticket.title }}</h5>
              <!-- Tampilkan tanggal dan harga -->
              <p class="card-text">
                <strong>Tanggal:</strong> {{ ticket.date }}<br />
                <strong>Harga:</strong> Rp {{ formatRupiah(ticket.price) }}
              </p>
            </div>
          </div>
        </div>
      </div>

      <!-- Tampilkan pesan jika tidak ada tiket ditemukan -->
      <div v-if="!loading && filteredTickets.length === 0" class="text-center text-muted">
        Tidak ada tiket ditemukan.
      </div>
    </div>
  </div>
</template>

<script>
export default {
  // Data yang digunakan dalam komponen
  data() {
    return {
      tickets: [],     // Data tiket dari backend
      search: '',      // Kata kunci untuk pencarian
      sortBy: '',      // Kriteria sortir ('date' atau 'price')
      loading: true,   // Status loading saat fetch data
      error: false     // Status error jika fetch gagal
    }
  },

  // Lifecycle hook saat komponen dimuat pertama kali
  async mounted() {
    try {
      // Ambil data dari backend (Cloudflare Worker)
      const res = await fetch("https://backend-tiket.finanurilaulia2104.workers.dev/tickets")
      if (!res.ok) throw new Error("Gagal mengambil data")
      // Simpan data ke array tickets
      this.tickets = await res.json()
    } catch (err) {
      // Jika error, tampilkan error di konsol dan aktifkan status error
      console.error(err)
      this.error = true
    } finally {
      // Setelah proses selesai (sukses/gagal), matikan loading
      this.loading = false
    }
  },

  // Property yang dihitung otomatis saat data berubah
  computed: {
    filteredTickets() {
      let result = this.tickets

      // Filter berdasarkan input pencarian
      if (this.search.trim()) {
        const keyword = this.search.toLowerCase()
        result = result.filter(ticket =>
          ticket.title.toLowerCase().includes(keyword)
        )
      }

      // Sortir berdasarkan tanggal jika dipilih
      if (this.sortBy === 'date') {
        result = result.slice().sort((a, b) => new Date(a.date) - new Date(b.date))
      }
      // Sortir berdasarkan harga jika dipilih
      else if (this.sortBy === 'price') {
        result = result.slice().sort((a, b) => a.price - b.price)
      }

      return result
    }
  },

  // Method yang bisa dipanggil di template
  methods: {
    // Format angka ke dalam format rupiah
    formatRupiah(value) {
      return Number(value).toLocaleString('id-ID')
    },
    // Fungsi toggle dark mode (hanya alert untuk saat ini)
    toggleDark() {
      alert('Mode gelap sudah aktif secara default 🎧')
    }
  }
}
</script>

<style scoped>
/* Warna border pink khusus untuk input dan select */
.border-pink {
  border-color: #f14ff1 !important;
}
</style>
