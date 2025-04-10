<script setup>
useHead({
  title: "IP",
  meta: [
    {
      name: "description",
      content: "IP",
    },
  ],
});

const supabase = useSupabaseClient();
const visitors = ref([]);
const editingVisitor = ref(null);
const isModalOpen = ref(false);
const isFooterEditOpen = ref(false);
const selectedVisitor = ref(null); // Visitor yang dipilih untuk diedit
const periodeData = ref(null); // Data periode dari database
const editingPeriode = ref(false); // Status edit periode
// Store footer totals in a reactive object
const footerTotals = ref({});

// Copy for editing
const editingFooter = ref(null);

// Fungsi untuk mengambil data alokasi
const getIp = async () => {
  const { data } = await supabase.from("ip").select("*").order("id", { ascending: true });
  if (data) {
    visitors.value = data;
  }
};

// Fungsi untuk mengambil data periode dari database
const getPeriode = async () => {
  try {
    const { data, error } = await supabase.from("periode").select("*").single();

    if (error) {
      throw error;
    }

    if (data) {
      periodeData.value = data;
    } else {
      // Jika tidak ada data, buat objek kosong
      periodeData.value = {
        judul: "PERIODE: TAHUN 2023/2024" // Default fallback
      };
    }
  } catch (error) {
    console.error("Error fetching periode data:", error.message);
    // Tetap sediakan nilai default jika terjadi error
    periodeData.value = {
      judul: "PERIODE: TAHUN 2023/2024"
    };
  }
};

// Fungsi untuk mengedit periode
const editPeriode = () => {
  editingPeriode.value = true;
};

// Fungsi untuk menyimpan perubahan periode
const savePeriodeChanges = async () => {
  try {
    if (!periodeData.value.judul || periodeData.value.judul.trim() === '') {
      alert("Judul periode tidak boleh kosong");
      return;
    }

    if (periodeData.value.id) {
      // Update data yang sudah ada
      const { error } = await supabase
        .from("periode")
        .update({ judul: periodeData.value.judul })
        .eq("id", periodeData.value.id);

      if (error) throw error;
    } else {
      // Insert data baru jika belum ada
      const { error } = await supabase
        .from("periode")
        .insert([{ judul: periodeData.value.judul }]);

      if (error) throw error;

      // Ambil data yang baru saja diinsert untuk mendapatkan ID
      await getPeriode();
    }

    editingPeriode.value = false;
  } catch (error) {
    console.error("Error saving periode data:", error.message);
    alert("Gagal menyimpan data periode: " + error.message);
  }
};

// Fungsi untuk mengedit data visitor
// const editVisitor = (visitor) => {
//   selectedVisitor.value = { ...visitor }; // Menyalin data visitor yang akan diedit
// };

// Fungsi untuk menyimpan perubahan setelah edit
const saveChanges = async () => {
  if (selectedVisitor.value) {
    const { data, error } = await supabase
      .from("ip")
      .update({
        nama_di: selectedVisitor.value.nama_di,
        mt_1: selectedVisitor.value.mt_1,
        mt_2: selectedVisitor.value.mt_2,
        mt_3: selectedVisitor.value.mt_3,
        jumlah: selectedVisitor.value.jumlah,
      })
      .eq("id", selectedVisitor.value.id); // Update berdasarkan ID visitor

    if (error) {
      console.error("Error updating ip:", error);
    } else {
      // Perbarui data di tabel dan tutup modal
      const index = visitors.value.findIndex(v => v.id === selectedVisitor.value.id);
      visitors.value[index] = selectedVisitor.value;
      selectedVisitor.value = null; // Tutup modal
    }
  }
};

// Fungsi untuk membatalkan edit periode
const cancelPeriodeEdit = () => {
  editingPeriode.value = false;
  // Reload data periode untuk memastikan data sesuai dengan database
  getPeriode();
};

// Menghitung jumlah dinamis dari Luas Areal, Realisasi, dan Minggu Ke 1 & 2
// const calculateTotal = () => {
//   const totalMt1 = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.mt_1 || 0), 0);
//   const totalMt2 = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.mt_2 || 0), 0);
//   const totalMt3 = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.mt_3 || 0), 0);
//   const totalJumlah = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.jumlah || 0), 0);

//   return { totalMt1, totalMt2, totalMt3, totalJumlah };
// };

const getFooterTotals = async () => {
  const { data, error } = await supabase.from("jml_ip").select("*").single(); // Fetch a single row from jml_iksi
  if (data) {
    footerTotals.value = data; // Update footerTotals with the actual data from jml_iksi table
  } else {
    console.error('Error fetching footer totals:', error);
  }
};

const openEditModal = (visitor) => {
  editingVisitor.value = { ...visitor };
  isModalOpen.value = true;
};

const openFooterEditModal = () => {
  editingFooter.value = { ...footerTotals.value };
  isFooterEditOpen.value = true;
};

const closeModal = () => {
  isModalOpen.value = false;
  editingVisitor.value = null;
};

const closeFooterEditModal = () => {
  isFooterEditOpen.value = false;
  editingFooter.value = null;
};

// const saveChanges = async () => {
//   if (!editingVisitor.value) return;

//   try {
//     const { error } = await supabase
//       .from("iksi")
//       .update(editingVisitor.value)
//       .eq("id", editingVisitor.value.id);

//     if (error) throw error;

//     // Update the local data immediately without refetching
//     const index = visitors.value.findIndex(v => v.id === editingVisitor.value.id);
//     if (index !== -1) {
//       // Directly modify the visitor data at the specified index
//       visitors.value[index] = { ...editingVisitor.value };

//       // Optionally sort the data if needed
//       visitors.value.sort((a, b) => a.id - b.id);
//     }

//     closeModal();
//   } catch (error) {
//     console.error("Error saving changes:", error);
//     alert("Gagal menyimpan perubahan. Silakan coba lagi.");
//   }
// };

const saveFooterTotals = async () => {
  if (!editingFooter.value) return;

  try {
    // Menyimpan perubahan footer totals ke database
    const { error } = await supabase
      .from("jml_iksi")  // Sesuaikan dengan tabel yang Anda miliki
      .upsert([editingFooter.value], { onConflict: ['id'] });  // asumsikan ada ID unik untuk footer totals

    if (error) throw error;

    // Update data lokal jika berhasil
    footerTotals.value = { ...editingFooter.value };
    closeFooterEditModal();
  } catch (error) {
    console.error("Error saving footer totals:", error);
    alert("Gagal menyimpan perubahan footer totals. Silakan coba lagi.");
  }
};

onMounted(() => {
  getIp();
  getPeriode();
  getFooterTotals(); // Ambil data periode saat komponen di-mount
});
</script>

<template>
  <div class="judul m-5 text-center">
    <h2>INDEK PERTANAMAN (IP)</h2>
    <div>
      <!-- Tampilkan judul periode dari database -->
      <div v-if="!editingPeriode">
        <h3>{{ periodeData?.judul }}</h3>
        <button @click="editPeriode" class="btn btn-primary">Edit Periode</button>
      </div>

      <!-- Form edit periode -->
      <div v-else class="periode-edit-form">
        <div class="form-group">
          <label for="judul">Judul Periode:</label>
          <input type="text" id="judul" v-model="periodeData.judul" class="form-control"
            placeholder="Masukkan judul periode" />
        </div>
        <div class="button-group">
          <button @click="savePeriodeChanges" class="btn btn-success">Simpan</button>
          <button @click="cancelPeriodeEdit" class="btn btn-danger">Batal</button>
        </div>
      </div>
    </div>
  </div>

  <div class="table-container">
    <table class="table table-bordered">
      <thead>
        <tr>
          <th scope="col">No</th>
          <th scope="col">Nama DI</th>
          <th scope="col">Masa Tanam 1</th>
          <th scope="col">Masa Tanam 2</th>
          <th scope="col">Masa Tanam 3</th>
          <th scope="col">Jumlah</th>
          <th scope="col">Aksi</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(visitor, i) in visitors" :key="i">
            <th scope="row">{{ i + 1 }}.</th>
            <td>{{ visitor.nama_di }}</td>
            <td class="text-end">{{ visitor.mt_1 }}</td>
            <td class="text-end">{{ visitor.mt_2 }}</td>
            <td class="text-end">{{ visitor.mt_3 }}</td>
            <td class="text-center">{{ visitor.jumlah }}</td>
            <td>
              <button @click="openEditModal(visitor)" class="btn btn-primary btn-sm" style="color:white;">
                Edit
              </button>
            </td>
          </tr>
        <tfoot>
          <tr class="table-light">
            <td colspan="3" class="text-center">Jumlah</td>
            <td class="text-end">{{ footerTotals.mt_1 }}</td>
            <td class="text-end">{{ footerTotals.mt_2 }}</td>
            <td class="text-end">{{ footerTotals.mt_3 }}</td>
            <td class="text-center">{{ footerTotals.jumlah }}</td>
            <td>
              <button @click="openFooterEditModal()" class="btn btn-primary btn-sm" style="color:white;">
                Edit
              </button>
            </td>
          </tr>
        </tfoot>
        <!-- <tr>
          <th scope="row"></th>
          <td><strong>Rata-rata</strong></td>
          <td>{{ calculateTotal().totalMt1.toFixed(2) }}</td> 
          <td>{{ calculateTotal().totalMt2.toFixed(2) }}</td> 
          <td>{{ calculateTotal().totalMt3.toFixed(2) }}</td> 
          <td>{{ calculateTotal().totalJumlah.toFixed(2) }}</td> <
          <td></td>
        </tr> -->
      </tbody>
    </table>
  </div>

  <!-- Edit Modal for Row Data -->
  <div v-if="isModalOpen" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <h5 class="modal-title">Edit Data</h5>
        <div v-if="editingVisitor" class="edit-form">
          <div class="form-group mb-2">
            <label for="nama_di">Nama Daerah Irigasi</label>
            <input v-model="editingVisitor.nama_di" type="text" id="nama_di" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="mt_1">Masa Tanam 1</label>
            <input v-model="editingVisitor.mt_1" type="text" id="mt_1" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="mt_2">Masa Tanam 2</label>
            <input v-model="editingVisitor.mt_2" type="text" id="mt_2" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="mt_3">Masa Tanam 3</label>
            <input v-model="editingVisitor.mt_3" type="text" id="mt_3" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="jumlah">Jumlah</label>
            <input v-model="editingVisitor.jumlah" type="text" id="jumlah" class="form-control" />
          </div>

          <div class="modal-footer mt-3">
            <button @click="closeModal" class="btn btn-secondary me-2">Batal</button>
            <button @click="saveChanges" class="btn btn-primary">Simpan</button>
          </div>
        </div>
      </div>
    </div>

  <!-- Edit Modal for Footer Totals -->
  <div v-if="isFooterEditOpen" class="modal-overlay" @click.self="closeFooterEditModal">
      <div class="modal-content">
        <h5 class="modal-title">Edit Jumlah Total</h5>
        <div v-if="editingFooter" class="edit-form">
          <div class="form-group mb-2">
            <label for="footer_mt_1">Masa Tanam 1</label>
            <input v-model="editingFooter.mt_1" type="text" id="footer_mt_1" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_mt_2">Masa Tanam 2</label>
            <input v-model="editingFooter.mt_2" type="text" id="footer_mt_2" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_mt_3">Masa Tanam 3</label>
            <input v-model="editingFooter.mt_3" type="text" id="footer_mt_3" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_jumlah">Jumlah</label>
            <input v-model="editingFooter.jumlah" type="text" id="footer_jumlah" class="form-control" />
          </div>

          <div class="modal-footer mt-3 ">
            <button @click="closeFooterEditModal" class="btn btn-secondary me-2">Batal</button>
            <button @click="saveFooterTotals" class="btn btn-primary">Simpan</button>
          </div>
        </div>
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

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-success {
  background-color: #28a745;
  color: white;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.periode-edit-form {
  max-width: 500px;
  margin: 0 auto;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 5px;
  background-color: #f9f9f9;
}

.form-group {
  margin-bottom: 15px;
}

.form-control {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.button-group {
  display: flex;
  justify-content: center;
  gap: 10px;
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