<template>
<!-- membuat halaman admin -->
  <!-- Container utama dengan background gelap dan teks terang -->
  <div class="container-fluid py-5" style="background-color: #0d0d0d; min-height: 100vh; color: #eee;">
    <div class="container">

      <!-- Judul Halaman Admin -->
      <h2 class="fw-bold mb-4" style="color: #f14ff1;">🛠️ Admin Tiket Konser</h2>

      <!-- Form Tambah/Edit Tiket -->
      <div class="card bg-dark border border-pink mb-4 text-light">
        <div class="card-body">

          <!-- Judul Form berdasarkan mode edit -->
          <h5 class="card-title">{{ editMode ? '✏️ Edit Tiket' : '➕ Tambah Tiket' }}</h5>

          <div class="row">
            <!-- Input Judul Tiket -->
            <div class="col-md-6 mb-3">
              <label>Judul Tiket</label>
              <input v-model="form.title" class="form-control bg-dark text-light border-pink" />
            </div>

            <!-- Input Tanggal -->
            <div class="col-md-6 mb-3">
              <label>Tanggal</label>
              <input v-model="form.date" type="date" class="form-control bg-dark text-light border-pink" />
            </div>

            <!-- Input Harga -->
            <div class="col-md-6 mb-3">
              <label>Harga</label>
              <input v-model="form.price" type="number" class="form-control bg-dark text-light border-pink" />
            </div>
          </div>

          <!-- Tombol Simpan (tambah/edit) -->
          <button @click="submitForm" class="btn btn-pink me-2">
            {{ editMode ? '💾 Simpan' : '📤 Simpan Tiket' }}
          </button>

          <!-- Tombol Batal Edit -->
          <button v-if="editMode" @click="cancelEdit" class="btn btn-secondary">❌ Batal</button>
        </div>
      </div>

      <!-- Daftar Tiket -->
      <div class="row">
        <!-- Menampilkan semua tiket dengan v-for -->
        <div class="col-md-4 mb-4" v-for="ticket in tickets" :key="ticket.id">
          <div class="card bg-secondary text-light shadow border-0 h-100">
            <div class="card-body">

              <!-- Judul Tiket -->
              <h5 class="card-title" style="color: #f14ff1;">🎵 {{ ticket.title }}</h5>

              <!-- Informasi Tanggal dan Harga -->
              <p class="card-text">
                <strong>Tanggal:</strong> {{ ticket.date }}<br />
                <strong>Harga:</strong> Rp {{ formatRupiah(ticket.price) }}
              </p>

              <!-- Tombol Edit & Hapus -->
              <button @click="editTicket(ticket)" class="btn btn-sm btn-outline-light me-2">✏️ Edit</button>
              <button @click="deleteTicket(ticket.id)" class="btn btn-sm btn-outline-danger">🗑️ Hapus</button>

            </div>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // Menyimpan daftar tiket
      tickets: [],

      // Data form untuk input tiket
      form: {
        title: '',
        date: '',
        price: ''
      },

      // Apakah sedang dalam mode edit
      editMode: false,

      // ID tiket yang sedang diedit
      editId: null
    }
  },

  // Saat komponen dimount, ambil daftar tiket
  async mounted() {
    this.fetchTickets()
  },

  methods: {
    // Ambil semua tiket dari backend API
    async fetchTickets() {
      const res = await fetch("https://backend-tiket.finanurilaulia2104.workers.dev/tickets")
      this.tickets = await res.json()
    },

    // Simpan data dari form ke backend (POST/PUT)
    async submitForm() {
      const method = this.editMode ? 'PUT' : 'POST'
      const url = this.editMode
        ? `https://backend-tiket.finanurilaulia2104.workers.dev/tickets/${this.editId}`
        : `https://backend-tiket.finanurilaulia2104.workers.dev/tickets`

      await fetch(url, {
        method,
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(this.form)
      })

      this.resetForm()     // Reset form setelah kirim
      this.fetchTickets()  // Refresh daftar tiket
    },

    // Aktifkan mode edit dan isi form dengan data tiket
    editTicket(ticket) {
      this.editMode = true
      this.editId = ticket.id
      this.form = {
        title: ticket.title,
        date: ticket.date,
        price: ticket.price
      }
    },

    // Batalkan edit dan reset form
    cancelEdit() {
      this.resetForm()
    },

    // Reset form ke default
    resetForm() {
      this.editMode = false
      this.editId = null
      this.form = { title: '', date: '', price: '' }
    },

    // Hapus tiket dengan konfirmasi
    async deleteTicket(id) {
      if (confirm("Yakin ingin menghapus tiket ini?")) {
        await fetch(`https://backend-tiket.finanurilaulia2104.workers.dev/tickets/${id}`, {
          method: "DELETE"
        })
        this.fetchTickets() // Refresh setelah hapus
      }
    },

    // Format angka ke mata uang Rupiah
    formatRupiah(value) {
      return Number(value).toLocaleString('id-ID')
    }
  }
}
</script>

<style scoped>
/* Border warna pink */
.border-pink {
  border-color: #f14ff1 !important;
}

/* Tombol dengan warna pink */
.btn-pink {
  background-color: #f14ff1;
  color: white;
  border: none;
}

/* Hover efek untuk tombol pink */
.btn-pink:hover {
  background-color: #d200d2;
}
</style>
