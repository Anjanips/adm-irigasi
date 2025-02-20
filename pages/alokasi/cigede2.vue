<script setup>
useHead({
  title: "Alokasi",
  meta: [
    {
      name: "description",
      content: "Alokasi",
    },
  ],
});

const supabase = useSupabaseClient();
const visitors = ref([]);
const selectedVisitor = ref(null); // Visitor yang dipilih untuk diedit

// Fungsi untuk mengambil data alokasi
const getAlokasi = async () => {
  const { data } = await supabase.from("alokasi_cigede").select("*").order("id", { ascending: true });
  if (data) {
    visitors.value = data;
  }
};

// Fungsi untuk mengedit data visitor
const editVisitor = (visitor) => {
  selectedVisitor.value = { ...visitor }; // Menyalin data visitor yang akan diedit
};

// Fungsi untuk menyimpan perubahan setelah edit
const saveChanges = async () => {
  if (selectedVisitor.value) {
    const { data, error } = await supabase
      .from("alokasi_cigede")
      .update({
        nama_petak: selectedVisitor.value.nama_petak,
        luas_areal: selectedVisitor.value.luas_areal,
        realisasi: selectedVisitor.value.realisasi,
        minggu_ke1: selectedVisitor.value.minggu_ke1,
        minggu_ke2: selectedVisitor.value.minggu_ke2,
      })
      .eq("id", selectedVisitor.value.id); // Update berdasarkan ID visitor

    if (error) {
      console.error("Error updating alokasi:", error);
    } else {
      // Perbarui data di tabel dan tutup modal
      const index = visitors.value.findIndex(v => v.id === selectedVisitor.value.id);
      visitors.value[index] = selectedVisitor.value;
      selectedVisitor.value = null; // Tutup modal
    }
  }
};

// Menghitung jumlah dinamis dari Luas Areal, Realisasi, dan Minggu Ke 1
const calculateTotal = () => {
  const totalLuas = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.luas_areal || 0), 0);
  const totalRealisasi = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.realisasi || 0), 0);
  const totalMingguKe1 = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.minggu_ke1 || 0), 0);
  const totalMingguKe2 = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.minggu_ke2 || 0), 0);

  return { totalLuas, totalRealisasi, totalMingguKe1, totalMingguKe2 };
};

onMounted(() => {
  getAlokasi();
});
</script>



<template>
  <div class="judul m-5 text-center">
    <h2>DI CIGEDE KAB/KOTA TASIKMALAYA </h2>
    <h3>PERIODE: TANGGAL 1 FEBRUARI s/d 15 FEBRUARI 2025</h3>
  </div>
  <div class="table-container">
    <table class="table table-bordered">
      <thead>
        <tr>
          <th scope="col">No</th>
          <th scope="col">Nama Petak Tersier</th>
          <th scope="col">Luas Areal (ha)</th>
          <th scope="col">Realisasi Areal</th>
          <th scope="col">Februari 2025, Minggu Ke 1</th>
          <th scope="col">Februari 2025, Minggu Ke 2</th>
          <th scope="col">Aksi</th> <!-- Kolom aksi untuk tombol edit -->
        </tr>
      </thead>
      <tbody>
        <tr v-for="(visitor, i) in visitors" :key="i">
          <th scope="row">{{ i + 1 }}.</th>
          <td>{{ visitor.nama_petak }}</td>
          <td>{{ visitor.luas_areal }}</td>
          <td>{{ visitor.realisasi }}</td>
          <td>{{ visitor.minggu_ke1 }}</td>
          <td>{{ visitor.minggu_ke2 }}</td>
          <td>
            <button @click="editVisitor(visitor)" class="btn btn-warning">Edit</button> <!-- Tombol edit -->
          </td>
        </tr>
        <tr>
          <th scope="row"></th>
          <td><strong>Jumlah Akhir</strong></td>
          <td>{{ calculateTotal().totalLuas.toFixed(2) }}</td> <!-- Menampilkan total luas areal -->
          <td>{{ calculateTotal().totalRealisasi.toFixed(2) }}</td> <!-- Menampilkan total realisasi -->
          <td>{{ calculateTotal().totalMingguKe1.toFixed(2) }}</td> <!-- Menampilkan total minggu ke-1 -->
          <td>{{ calculateTotal().totalMingguKe2.toFixed(2) }}</td> <!-- Menampilkan total minggu ke-1 -->
          <td></td>
        </tr>
      </tbody>
    </table>
  </div>

  <!-- Modal untuk edit data visitor -->
  <div v-if="selectedVisitor" class="modal">
    <div class="modal-content">
      <h2>Edit Data Alokasi</h2>
      <form @submit.prevent="saveChanges">
        <div>
          <label for="nama_petak">Nama Petak Tersier:</label>
          <input type="text" v-model="selectedVisitor.nama_petak" />
        </div>
        <div>
          <label for="luas_areal">Luas Areal (ha):</label>
          <input type="text" v-model="selectedVisitor.luas_areal" />
        </div>
        <div>
          <label for="realisasi">Realisasi Areal:</label>
          <input type="text" v-model="selectedVisitor.realisasi" />
        </div>
        <div>
          <label for="minggu_ke1">Februari 2025, Minggu Ke 1:</label>
          <input type="text" v-model="selectedVisitor.minggu_ke1" />
        </div>
        <div>
          <label for="minggu_ke1">Februari 2025, Minggu Ke 2:</label>
          <input type="text" v-model="selectedVisitor.minggu_ke2" />
        </div>
        <button type="submit">Simpan</button>
        <button type="button" @click="selectedVisitor = null">Batal</button>
      </form>
    </div>
  </div>
</template>


<style scoped>
.table-container {
  width: 90%;
  margin: 0 auto;
  font-size: 0.9rem;
}

.table td,
.table th {
  padding: 0.5rem;
  vertical-align: middle;
}

button {
  padding: 8px 16px;
  margin: 5px;
  cursor: pointer;
}

button[type="submit"] {
  background-color: green;
  color: white;
}

button[type="button"] {
  background-color: red;
  color: white;
}

/* Gaya untuk modal */
.modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background-color: white;
  padding: 20px;
  border-radius: 8px;
  width: 400px;
}

.modal-content input {
  width: 100%;
  padding: 8px;
  margin: 10px 0;
}
</style>
