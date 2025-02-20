<script setup>
useHead({
  title: "Iksi",
  meta: [
    {
      name: "description",
      content: "Iksi",
    },
  ],
});

const supabase = useSupabaseClient();
const visitors = ref([]);
const editingVisitor = ref(null);
const isModalOpen = ref(false);
const isFooterEditOpen = ref(false);

// Store footer totals in a reactive object
const footerTotals = ref({});

// Copy for editing
const editingFooter = ref(null);

const getIksi = async () => {
  const { data } = await supabase.from("iksi").select("*");
  if (data) visitors.value = data;
};

const getFooterTotals = async () => {
  const { data, error } = await supabase.from("jml_iksi").select("*").single(); // Fetch a single row from jml_iksi
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

const saveChanges = async () => {
  if (!editingVisitor.value) return;

  try {
    const { error } = await supabase
      .from("iksi")
      .update(editingVisitor.value)
      .eq("id", editingVisitor.value.id);

    if (error) throw error;

    // Update the local data immediately without refetching
    const index = visitors.value.findIndex(v => v.id === editingVisitor.value.id);
    if (index !== -1) {
      // Directly modify the visitor data at the specified index
      visitors.value[index] = { ...editingVisitor.value };

      // Optionally sort the data if needed
      visitors.value.sort((a, b) => a.id - b.id);
    }

    closeModal();
  } catch (error) {
    console.error("Error saving changes:", error);
    alert("Gagal menyimpan perubahan. Silakan coba lagi.");
  }
};

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
  getIksi();
  getFooterTotals(); // Fetch footer totals when the component is mounted
});
</script>

<template>
  <div class="">
    <!-- Table Content -->
    <h1 class="text-center h4 font-weight-bold mb-4">REKAP IKSI TAHUN 2024</h1>
    <h2 class="text-center h5 mb-6">UPTD PSDA WILAYAH SUNGAI CITANDUY</h2>
    <div class="table-container">
      <table class="table table-bordered">
        <thead class="table-success">
          <tr>
            <th class="text-center">NO.</th>
            <th class="text-center">NAMA DAERAH IRIGASI</th>
            <th class="text-center">KABUPATEN / KOTA</th>
            <th class="text-center">LUAS AREAL (Ha)</th>
            <th class="text-center">Prasarana Fisik</th>
            <th class="text-center">Produktivitas (Padi)</th>
            <th class="text-center">Sarana Penunjang</th>
            <th class="text-center">Organisasi Personalia</th>
            <th class="text-center">Dokumentasi</th>
            <th class="text-center">P3A/GP3A/IP3A</th>
            <th class="text-center">Jumlah</th>
            <th class="text-center">Aksi</th>
          </tr>
        </thead>
        <tbody>
          <tr class="table-light">
            <td colspan="12" class="text-center">VI UPTD PSDA WS.CITANDUY</td>
          </tr>

          <tr v-for="(visitor, i) in visitors" :key="i">
            <th scope="row">{{ i + 1 }}.</th>
            <td>{{ visitor.nama_daerah }}</td>
            <td>{{ visitor.kabupaten }}</td>
            <td class="text-end">{{ visitor.luas_areal }}</td>
            <td class="text-end">{{ visitor.prasarana }}</td>
            <td class="text-end">{{ visitor.produktivitas }}</td>
            <td class="text-end">{{ visitor.sarana_penunjang }}</td>
            <td class="text-end">{{ visitor.organisasi }}</td>
            <td class="text-end">{{ visitor.dokumentasi }}</td>
            <td class="text-end">{{ visitor.p3a }}</td>
            <td class="text-center">{{ visitor.jumlah }}</td>
            <td>
              <button @click="openEditModal(visitor)" class="btn btn-primary btn-sm" style="color:white;">
                Edit
              </button>
            </td>
          </tr>
        </tbody>
        <tfoot>
          <tr class="table-light">
            <td colspan="3" class="text-center">Jumlah</td>
            <td class="text-end">{{ footerTotals.luas_areal }}</td>
            <td class="text-end">{{ footerTotals.prasarana }}</td>
            <td class="text-end">{{ footerTotals.produktivitas }}</td>
            <td class="text-end">{{ footerTotals.sarana_penunjang }}</td>
            <td class="text-end">{{ footerTotals.organisasi }}</td>
            <td class="text-end">{{ footerTotals.dokumentasi }}</td>
            <td class="text-end">{{ footerTotals.p3a }}</td>
            <td class="text-center">{{ footerTotals.jumlah }}</td>
            <td>
              <button @click="openFooterEditModal()" class="btn btn-primary btn-sm" style="color:white;">
                Edit
              </button>
            </td>
          </tr>
        </tfoot>
      </table>
    </div>

    <!-- Edit Modal for Row Data -->
    <div v-if="isModalOpen" class="modal-overlay" @click.self="closeModal">
      <div class="modal-content">
        <h5 class="modal-title">Edit Data</h5>
        <div v-if="editingVisitor" class="edit-form">
          <div class="form-group mb-2">
            <label for="nama_daerah">Nama Daerah Irigasi</label>
            <input v-model="editingVisitor.nama_daerah" type="text" id="nama_daerah" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="kabupaten">Kabupaten / Kota</label>
            <input v-model="editingVisitor.kabupaten" type="text" id="kabupaten" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="luas_areal">Luas Areal (Ha)</label>
            <input v-model="editingVisitor.luas_areal" type="text" id="luas_areal" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="prasarana">Prasarana Fisik</label>
            <input v-model="editingVisitor.prasarana" type="text" id="prasarana" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="produktivitas">Produktivitas (Padi)</label>
            <input v-model="editingVisitor.produktivitas" type="text" id="produktivitas" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="sarana_penunjang">Sarana Penunjang</label>
            <input v-model="editingVisitor.sarana_penunjang" type="text" id="sarana_penunjang" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="organisasi">Organisasi Personalia</label>
            <input v-model="editingVisitor.organisasi" type="text" id="organisasi" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="dokumentasi">Dokumentasi</label>
            <input v-model="editingVisitor.dokumentasi" type="text" id="dokumentasi" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="p3a">P3A/GP3A/IP3A</label>
            <input v-model="editingVisitor.p3a" type="text" id="p3a" class="form-control" />
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
            <label for="footer_luas_areal">Luas Areal (Ha)</label>
            <input v-model="editingFooter.luas_areal" type="text" id="footer_luas_areal" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_prasarana">Prasarana Fisik</label>
            <input v-model="editingFooter.prasarana" type="text" id="footer_prasarana" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_produktivitas">Produktivitas (Padi)</label>
            <input v-model="editingFooter.produktivitas" type="text" id="footer_produktivitas" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_sarana_penunjang">Sarana Penunjang</label>
            <input v-model="editingFooter.sarana_penunjang" type="text" id="footer_sarana_penunjang"
              class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_organisasi">Organisasi Personalia</label>
            <input v-model="editingFooter.organisasi" type="text" id="footer_organisasi" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_dokumentasi">Dokumentasi</label>
            <input v-model="editingFooter.dokumentasi" type="text" id="footer_dokumentasi" class="form-control" />
          </div>
          <div class="form-group mb-2">
            <label for="footer_p3a">P3A/GP3A/IP3A</label>
            <input v-model="editingFooter.p3a" type="text" id="footer_p3a" class="form-control" />
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

/* Modal styles */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
  margin-top: 5%;
}

.modal-content {
  background: white;
  padding: 1.5rem;
  border-radius: 8px;
  width: 400px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
}

.edit-form .form-group {
  margin-bottom: 1rem;
}

.edit-form .form-group label {
  font-weight: bold;
}
</style>
