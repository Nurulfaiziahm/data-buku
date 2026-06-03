<template>
  <div class="app">
    <header class="hero">
      <h1>📚 Perpustakaan Universitas Terbuka</h1>
      <p>
        Sistem informasi untuk pengelolaan data buku, kategori, dan penempatan
        rak perpustakaan.
      </p>

      <div class="stats-grid">
        <div class="stat-card">
          <h2>{{ books.length }}</h2>
          <span>Total Buku</span>
        </div>
        <div class="stat-card">
          <h2>{{ kategoriCount }}</h2>
          <span>Kategori</span>
        </div>
        <div class="stat-card">
          <h2>{{ allRacks.length }}</h2>
          <span>Total Rak</span>
        </div>
        <div class="stat-card">
          <h2>{{ emptyRacks }}</h2>
          <span>Rak Kosong</span>
        </div>
      </div>
    </header>

    <button class="rack-toggle" @click="showRack = !showRack">
      🗄️ {{ showRack ? "Sembunyikan" : "Lihat" }} Status Rak
    </button>

    <section v-if="showRack" class="rack-panel">
      <h3>Status Rak Perpustakaan</h3>
      <div class="rack-grid">
        <div v-for="rack in allRacks" :key="rack.kode" class="rack-card">
          <strong>{{ rack.kode }}</strong>
          <span>{{ rack.kategori }}</span>
          <small>{{ rackUsage(rack.kode) }}/20 buku</small>
        </div>
      </div>
    </section>

    <section class="toolbar">
      <input v-model="search" placeholder="🔍 Cari judul buku..." />
      <select v-model="selectedCategory">
        <option>Semua</option>
        <option>Teknologi</option>
        <option>Pendidikan</option>
        <option>Ekonomi</option>
        <option>Hukum</option>
        <option>Novel</option>
      </select>
      <button @click="openAddModal">+ Tambah Buku</button>
    </section>

    <section class="books-grid">
      <div v-for="book in filteredBooks" :key="book.kodeBuku" class="book-card">
        <span class="badge">{{ book.kategori }}</span>
        <h3>{{ book.judulBuku }}</h3>
        <p><b>Kode:</b> {{ book.kodeBuku }}</p>
        <p><b>Pengarang:</b> {{ book.pengarang }}</p>
        <p><b>Rak:</b> {{ book.lokasiRak }}</p>
        <p><b>Jumlah:</b> {{ book.jumlahBuku }}</p>
        <p><b>Tahun:</b> {{ book.tahunTerbit }}</p>
        <div class="actions">
          <button @click="editBook(book)">Edit</button>
          <button class="danger" @click="deleteBook(book.kodeBuku)">
            Hapus
          </button>
        </div>
      </div>
    </section>

    <div v-if="showModal" class="modal-backdrop">
      <div class="modal">
        <div v-if="showModal" class="modal-backdrop">
          <div class="modal">
            <h2>{{ editMode ? "Edit Buku" : "Tambah Buku" }}</h2>

            <div class="row">
              <input v-model="form.kodeBuku" placeholder="Kode Buku" />
              <input v-model="form.judulBuku" placeholder="Judul Buku" />
            </div>

            <div class="row">
              <input v-model="form.pengarang" placeholder="Pengarang" />
              <input v-model="form.penerbit" placeholder="Penerbit" />
            </div>

            <div class="row">
              <input
                type="number"
                min="0"
                max="2026"
                v-model="form.tahunTerbit"
                placeholder="Tahun Terbit (0 = Tidak Diketahui)"
              />
              <input
                type="number"
                v-model="form.jumlahBuku"
                placeholder="Jumlah Buku"
              />
            </div>

            <div class="row">
              <select v-model="form.kategori">
                <option>Teknologi</option>
                <option>Pendidikan</option>
                <option>Ekonomi</option>
                <option>Hukum</option>
                <option>Novel</option>
              </select>

              <select v-model="form.lokasiRak">
                <option value="">Pilih Rak</option>

                <option
                  v-for="rack in racksByCategory"
                  :key="rack.kode"
                  :value="rack.kode"
                >
                  {{ rack.kode }}
                </option>
              </select>
            </div>

            <div class="actions">
              <button @click="saveBook">Simpan</button>

              <button class="danger" @click="showModal = false">Batal</button>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <footer class="footer">
    <p>Sistem Informasi Perpustakaan</p>
    <p>By Nurul Faiziah M</p>
    <p>© 2026</p>
  </footer>
</template>

<script setup>
import { ref, reactive, computed, watch } from "vue";

const allRacks = [
  ...Array.from({ length: 5 }, (_, i) => ({
    kode: `T-0${i + 1}`,
    kategori: "Teknologi",
  })),
  ...Array.from({ length: 5 }, (_, i) => ({
    kode: `P-0${i + 1}`,
    kategori: "Pendidikan",
  })),
  ...Array.from({ length: 5 }, (_, i) => ({
    kode: `E-0${i + 1}`,
    kategori: "Ekonomi",
  })),
  ...Array.from({ length: 5 }, (_, i) => ({
    kode: `H-0${i + 1}`,
    kategori: "Hukum",
  })),
  ...Array.from({ length: 5 }, (_, i) => ({
    kode: `N-0${i + 1}`,
    kategori: "Novel",
  })),
];

const defaultBooks = [
  {
    kodeBuku: "BK001",
    judulBuku: "Algoritma Pemrograman",
    pengarang: "Abdul Kadir",
    penerbit: "Andi",
    tahunTerbit: 2023,
    kategori: "Teknologi",
    jumlahBuku: 10,
    lokasiRak: "T-01",
  },
  {
    kodeBuku: "BK002",
    judulBuku: "Strategi Pembelajaran",
    pengarang: "Syaiful Bahri",
    penerbit: "Rajawali",
    tahunTerbit: 2022,
    kategori: "Pendidikan",
    jumlahBuku: 8,
    lokasiRak: "P-01",
  },
  {
    kodeBuku: "BK003",
    judulBuku: "Pengantar Ekonomi",
    pengarang: "Sadono Sukirno",
    penerbit: "Prenada",
    tahunTerbit: 2021,
    kategori: "Ekonomi",
    jumlahBuku: 6,
    lokasiRak: "E-01",
  },
  {
    kodeBuku: "BK004",
    judulBuku: "Hukum Bisnis",
    pengarang: "Zaeni",
    penerbit: "RajaGrafindo",
    tahunTerbit: 2023,
    kategori: "Hukum",
    jumlahBuku: 7,
    lokasiRak: "H-01",
  },
  {
    kodeBuku: "BK005",
    judulBuku: "Laskar Pelangi",
    pengarang: "Andrea Hirata",
    penerbit: "Bentang",
    tahunTerbit: 2020,
    kategori: "Novel",
    jumlahBuku: 12,
    lokasiRak: "N-01",
  },
];

const books = ref(
  JSON.parse(localStorage.getItem("library-final")) || defaultBooks,
);
watch(
  books,
  () => localStorage.setItem("library-final", JSON.stringify(books.value)),
  { deep: true },
);

const search = ref("");
const selectedCategory = ref("Semua");
const showModal = ref(false);
const showRack = ref(false);
const editMode = ref(false);
const oldKode = ref("");

const form = reactive({
  kodeBuku: "",
  judulBuku: "",
  pengarang: "",
  penerbit: "",
  tahunTerbit: "",
  kategori: "Teknologi",
  jumlahBuku: "",
  lokasiRak: "",
});

const filteredBooks = computed(() =>
  books.value.filter(
    (b) =>
      (selectedCategory.value === "Semua" ||
        b.kategori === selectedCategory.value) &&
      b.judulBuku.toLowerCase().includes(search.value.toLowerCase()),
  ),
);
const kategoriCount = computed(
  () => new Set(books.value.map((b) => b.kategori)).size,
);
const emptyRacks = computed(
  () => allRacks.filter((r) => rackUsage(r.kode) === 0).length,
);
const racksByCategory = computed(() =>
  allRacks.filter((r) => r.kategori === form.kategori),
);

function rackUsage(kode) {
  return books.value
    .filter((b) => b.lokasiRak === kode)
    .reduce((a, b) => a + Number(b.jumlahBuku), 0);
}
function openAddModal() {
  editMode.value = false;
  Object.assign(form, {
    kodeBuku: "",
    judulBuku: "",
    pengarang: "",
    penerbit: "",
    tahunTerbit: "",
    kategori: "Teknologi",
    jumlahBuku: "",
    lokasiRak: "",
  });
  showModal.value = true;
}
function editBook(book) {
  editMode.value = true;
  oldKode.value = book.kodeBuku;
  Object.assign(form, { ...book });
  showModal.value = true;
}
function saveBook() {
  if (
    !form.kodeBuku ||
    !form.judulBuku ||
    !form.pengarang ||
    !form.penerbit ||
    form.tahunTerbit === "" ||
    !form.kategori ||
    !form.lokasiRak ||
    !form.jumlahBuku
  ) {
    return alert("Semua data wajib diisi");
  }
  const tahun = Number(form.tahunTerbit);
  const tahunSekarang = new Date().getFullYear();

  if (tahun !== 0 && (tahun < 1900 || tahun > tahunSekarang)) {
    return alert(`Tahun harus 0 atau antara 1900-${tahunSekarang}`);
  }
  const cap = 20;
  if (
    rackUsage(form.lokasiRak) + Number(form.jumlahBuku) > cap &&
    !editMode.value
  ) {
    return alert("Rak penuh (maks 20 buku)");
  }
  if (editMode.value) {
    const i = books.value.findIndex((b) => b.kodeBuku === oldKode.value);
    books.value[i] = { ...form };
    alert("Data buku berhasil diperbarui");
  } else {
    books.value.unshift({ ...form });

    alert("Data buku berhasil ditambahkan");
  }
  showModal.value = false;
}
function deleteBook(k) {
  if (!confirm("Hapus buku?")) return;
  books.value = books.value.filter((b) => b.kodeBuku !== k);
  alert("Data buku berhasil dihapus");
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}
.app {
  padding: 24px;
  background: #f4f7fb;
  min-height: 100vh;
}
.hero {
  background: linear-gradient(135deg, #2563eb, #4f46e5);
  padding: 40px;
  color: #fff;
  border-radius: 24px;
  text-align: center;
}
.stats-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(180px, 1fr));
  gap: 15px;
  margin-top: 20px;
}
.stat-card {
  background: rgba(255, 255, 255, 0.15);
  padding: 10px;
  border-radius: 16px;
}
.stat-card h2 {
  margin: 0;
  font-size: 1.5rem;
}
.stat-card span {
  display: block;
  margin-top: 8px;
}
.rack-panel,
.book-card {
  background: #fff;
}
.rack-toggle {
  background: #0f172a;
  color: white;
  border: none;
  padding: 12px 18px;
  border-radius: 12px;
  cursor: pointer;
  margin-top: 20px;
  margin-bottom: 0;
}
.rack-panel {
  margin: 20px 0;
  padding: 20px;
  border-radius: 20px;
}
.rack-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(180px, 1fr));
  gap: 16px;
  margin-top: 20px;
}
.rack-card {
  background: white;
  border: 1px solid #e5e7eb;
  border-radius: 16px;
  padding: 16px;
  text-align: left;
  min-height: 120px;

  display: flex;
  flex-direction: column;
  justify-content: space-between;

  transition: 0.2s;
}
.rack-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.08);
}
.toolbar {
  display: flex;
  gap: 10px;
  margin: 20px 0;
  flex-wrap: wrap;
}
.toolbar input {
  flex: 1;
  min-width: 250px;
}
input,
select {
  box-sizing: border-box;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 10px;
}
button {
  background: #2563eb;
  color: #fff;
  border: none;
  padding: 12px 16px;
  border-radius: 10px;
  cursor: pointer;
}
.books-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 16px;
}
.book-card {
  padding: 18px;
  border-radius: 18px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.08);
}
.book-card:hover {
  transform: translateY(-4px);
}
.badge {
  background: #2563eb;
  color: #fff;
  padding: 4px 10px;
  border-radius: 999px;
}
.actions {
  display: flex;
  gap: 8px;
  margin-top: 12px;
}
.danger {
  background: #dc2626;
}
.modal-backdrop {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}
.modal {
  background: #fff;
  padding: 24px;
  border-radius: 24px;
  width: 700px;
  max-width: 90%;
}
.row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin: 10px 0;
}
.modal h2 {
  margin-bottom: 20px;
}
.modal .actions {
  display: flex;
  gap: 10px;
  margin-top: 20px;
}
.modal .actions button {
  flex: 1;
}
.footer {
  text-align: center;
  margin-top: 20px;
  padding: 5px;
  color: #64748b;
  font-size: 14px;
  line-height: 1.4;
}
/* =========================
   MOBILE RESPONSIVE
========================= */
@media (max-width: 768px) {
  .app {
    padding: 12px;
  }
  .hero {
    padding: 24px 16px;
  }
  .hero h1 {
    font-size: 1.6rem;
    line-height: 1.3;
  }
  .hero p {
    font-size: 0.9rem;
  }
  .toolbar {
    flex-direction: column;
  }
  .toolbar input,
  .toolbar select,
  .toolbar button {
    width: 100%;
  }
  .books-grid {
    grid-template-columns: 1fr;
  }
  .rack-grid {
    grid-template-columns: 1fr;
  }
  .row {
    grid-template-columns: 1fr;
  }
  .actions {
    flex-direction: column;
  }
  .actions button {
    width: 100%;
  }
  .modal {
    width: 95%;
    max-width: none;
    padding: 18px;
  }
}
@media (max-width: 480px) {
  .stats-grid {
    grid-template-columns: 1fr;
  }
  .hero h1 {
    font-size: 1.3rem;
  }
}
</style>
