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
const periodeData = ref([]);


// Fungsi untuk mengambil data alokasi
const getIp = async () => {
  const { data } = await supabase.from("ip").select("*").order("id", { ascending: true });
  if (data) {
    visitors.value = data;
  }
};

// Fungsi untuk mengambil data periode dari database
const getPeriode = async () => {

  const { data, error } = await supabase.from("periode").select("*")
  if (data) {
    periodeData.value = data;
  };
};


// Menghitung jumlah dinamis dari Luas Areal, Realisasi, dan Minggu Ke 1 & 2
const calculateTotal = () => {
  const totalNamaDi = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.nama_di || 0), 0);
  const totalMt1 = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.mt_1 || 0), 0);
  const totalMt2 = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.mt_2 || 0), 0);
  const totalMt3 = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.mt_3 || 0), 0);
  const totalJumlah = visitors.value.reduce((acc, visitor) => acc + parseFloat(visitor.jumlah || 0), 0);

  return { totalNamaDi, totalMt1, totalMt2, totalMt3, totalJumlah };
};

onMounted(() => {
  getIp();
  getPeriode();
});
</script>


<template>
  <div class="judul m-5 text-center">
    <h2>INDEK PERTANAMAN (IP)</h2>
    <div v-for="(periode, i) in periodeData" :key="i">
      <h3>{{ periode.judul }}</h3>
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
        </tr>
      </thead>
      <tbody>
        <tr v-for="(visitor, i) in visitors" :key="i">
          <th scope="row">{{ i + 1 }}.</th>
          <td>{{ visitor.nama_di }}</td>
          <td>{{ visitor.mt_1 }}</td>
          <td>{{ visitor.mt_2 }}</td>
          <td>{{ visitor.mt_3 }}</td>
          <td>{{ visitor.jumlah }}</td>
        </tr>
        <tr>
          <th scope="row"></th>
          <td><strong>Rata-rata</strong></td>
          <td>{{ calculateTotal().totalNamaDi.toFixed(2) }}</td> <!-- Menampilkan total nama di -->
          <td>{{ calculateTotal().totalMt1.toFixed(2) }}</td> <!-- Menampilkan total masa tanam 1 -->
          <td>{{ calculateTotal().totalMt2.toFixed(2) }}</td> <!-- Menampilkan total masa tanam 2 -->
          <td>{{ calculateTotal().totalMt3.toFixed(2) }}</td> <!-- Menampilkan total masa tanam 3 -->
          <td>{{ calculateTotal().totalJumlah.toFixed(2) }}</td> <!-- Menampilkan total jumlah -->
        </tr>

      </tbody>
    </table>
  </div>
</template>

<style scoped>
.table-container {
  width: 90%;
  margin: 0 auto;
  font-size: 0.9rem;
}

.table img {
  width: 80px;
  /* Set fixed width for images */
  height: auto;
  object-fit: cover;
}

.table td,
.table th {
  padding: 0.5rem;
  vertical-align: middle;
}

.btn {
  margin-left: 5%;

}
</style>