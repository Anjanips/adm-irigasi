<script setup>
useHead({
    title: "Tanam",
    meta: [
        {
            name: "description",
            content: "Tanam",
        },
    ],
});

const supabase = useSupabaseClient();
const visitors = ref([]);
const selectedVisitor = ref(null);
const periodeData = ref(null); // Data periode dari database
const editingPeriode = ref(false); // Status edit periode

// Fungsi untuk mengambil data tanam dari Supabase
const getTanam = async () => {
    const { data, error } = await supabase.from("tanam").select("*");
    if (data) {
        // Menambahkan URL gambar ke setiap visitor
        visitors.value = data.map(visitor => {
            const imageUrl = supabase.storage
                .from('img') // Ganti dengan nama bucket Anda
                .getPublicUrl(visitor.img); // Ganti dengan nama kolom yang menyimpan path gambar

            return { ...visitor, imgUrl: imageUrl.data.publicUrl };
        });
    } else {
        console.error('Error fetching tanam data:', error);
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
                judul: "PERIODE: TANGGAL 1 FEBRUARI s/d 15 FEBRUARI 2025" // Default fallback
            };
        }
    } catch (error) {
        console.error("Error fetching periode data:", error.message);
        // Tetap sediakan nilai default jika terjadi error
        periodeData.value = {
            judul: "PERIODE: TANGGAL 1 FEBRUARI s/d 15 FEBRUARI 2025"
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

// Fungsi untuk membatalkan edit periode
const cancelPeriodeEdit = () => {
    editingPeriode.value = false;
    // Reload data periode untuk memastikan data sesuai dengan database
    getPeriode();
};

// Fungsi untuk mengedit visitor
const editVisitor = (visitor) => {
    selectedVisitor.value = { ...visitor }; // Salin data visitor untuk diedit
};

// Fungsi untuk menyimpan perubahan setelah edit
const saveChanges = async () => {
    if (selectedVisitor.value) {
        const { data, error } = await supabase
            .from("tanam")
            .update({
                nama_bendung: selectedVisitor.value.nama_bendung,
                luas_layanan: selectedVisitor.value.luas_layanan,
                faktork: selectedVisitor.value.faktork,
                pengolahan: selectedVisitor.value.pengolahan,
                pertumbuhan: selectedVisitor.value.pertumbuhan,
                panen: selectedVisitor.value.panen,
                palawija: selectedVisitor.value.palawija,
                bera: selectedVisitor.value.bera,
                kekeringan: selectedVisitor.value.kekeringan,
                banjir: selectedVisitor.value.banjir,
                puso: selectedVisitor.value.puso,
                mt: selectedVisitor.value.mt,
            })
            .eq("id", selectedVisitor.value.id); // Update berdasarkan ID visitor

        if (error) {
            console.error("Error updating visitor:", error);
        } else {
            // Jika berhasil, perbarui data di tabel dan tutup modal
            const index = visitors.value.findIndex(v => v.id === selectedVisitor.value.id);
            visitors.value[index] = selectedVisitor.value;
            selectedVisitor.value = null; // Tutup modal setelah berhasil
        }
    }
};

onMounted(() => {
    getPeriode();
    getTanam();
});
</script>

<template>
    <div class="judul m-5 text-center">
        <h2>REALISASI TANAM</h2>
        <div v-if="!editingPeriode">
            <h3>{{ periodeData?.judul }}</h3>
            <button @click="editPeriode" class="btn btn-primary">Edit Periode</button>
        </div>

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


    <div class="table-container">
        <table class="table table-bordered p-5">
            <thead>
                <tr class="table-success text-center">
                    <th>No</th>
                    <th>Bendung</th>
                    <th>Nama Bendung</th>
                    <th>Luas Layanan (Ha)</th>
                    <th>Faktor K</th>
                    <th>Pengolahan (Ha)</th>
                    <th>Pertumbuhan (Ha)</th>
                    <th>Panen (Ha)</th>
                    <th>Palawija (Ha)</th>
                    <th>Bera (Ha)</th>
                    <th>Kekeringan (Ha)</th>
                    <th>Banjir (Ha)</th>
                    <th>Puso (Ha)</th>
                    <th>Mulai Tanam (bln)</th>
                    <th>Aksi</th>
                </tr>
            </thead>

            <tbody>
                <tr v-for="(visitor, i) in visitors" :key="i">
                    <th scope="row">{{ i + 1 }}.</th>
                    <td><img :src="visitor.img" alt="Visitor Image" width="100" height="100" /></td>
                    <td>{{ visitor.nama_bendung }}</td>
                    <td>{{ visitor.luas_layanan }}</td>
                    <td>{{ visitor.faktork }}</td>
                    <td>{{ visitor.pengolahan }}</td>
                    <td>{{ visitor.pertumbuhan }}</td>
                    <td>{{ visitor.panen }}</td>
                    <td>{{ visitor.palawija }}</td>
                    <td>{{ visitor.bera }}</td>
                    <td>{{ visitor.kekeringan }}</td>
                    <td>{{ visitor.banjir }}</td>
                    <td>{{ visitor.puso }}</td>
                    <td>{{ visitor.mt }}</td>
                    <td>
                        <button @click="editVisitor(visitor)" class="btn btn-primary">Edit</button> <!-- Tombol edit -->
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

    <!-- Modal untuk edit data visitor -->
    <div v-if="selectedVisitor" class="modal">
        <div class="modal-content">
            <h2>Edit Data</h2>
            <form @submit.prevent="saveChanges">
                <div>
                    <label for="nama_bendung">Nama Bendung:</label>
                    <input type="text" v-model="selectedVisitor.nama_bendung" />
                </div>
                <div>
                    <label for="luas_layanan">Luas Layanan:</label>
                    <input type="text" v-model="selectedVisitor.luas_layanan" />
                </div>
                <div>
                    <label for="faktork">Faktor K:</label>
                    <input type="text" v-model="selectedVisitor.faktork" />
                </div>
                <div>
                    <label for="pengolahan">Pengolahan:</label>
                    <input type="text" v-model="selectedVisitor.pengolahan" />
                </div>
                <div>
                    <label for="pertumbuhan">Pertumbuhan:</label>
                    <input type="text" v-model="selectedVisitor.pertumbuhan" />
                </div>
                <div>
                    <label for="panen">Panen:</label>
                    <input type="text" v-model="selectedVisitor.panen" />
                </div>
                <div>
                    <label for="palawija">Palawija:</label>
                    <input type="text" v-model="selectedVisitor.palawija" />
                </div>
                <div>
                    <label for="bera">Bera:</label>
                    <input type="text" v-model="selectedVisitor.bera" />
                </div>
                <div>
                    <label for="kekeringan">Kekeringan:</label>
                    <input type="text" v-model="selectedVisitor.kekeringan" />
                </div>
                <div>
                    <label for="banjir">Banjir:</label>
                    <input type="text" v-model="selectedVisitor.banjir" />
                </div>
                <div>
                    <label for="puso">Puso:</label>
                    <input type="text" v-model="selectedVisitor.puso" />
                </div>
                <div>
                    <label for="mt">Mulai Tanam:</label>
                    <input type="text" v-model="selectedVisitor.mt" />
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

.table img {
    width: 100px;
    height: 100px;
    object-fit: cover;
}

.table td,
.table th {
    padding: 0.5rem;
    vertical-align: middle;
}

/* Gaya untuk modal */
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
    width: 350px;
    /* Lebar modal lebih kecil */
    max-height: 80vh;
    /* Batas tinggi modal agar tidak melebihi 80% dari viewport */
    overflow-y: auto;
    /* Menambahkan scroll vertikal jika konten lebih banyak */
}

/* Pastikan input memiliki padding dan margin yang baik */
.modal-content input {
    width: 100%;
    padding: 8px;
    margin: 10px 0;
    box-sizing: border-box;
}

/* Styling untuk tombol */
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
</style>
