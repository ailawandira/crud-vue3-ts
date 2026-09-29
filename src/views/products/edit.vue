<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRoute, useRouter } from "vue-router";
import Api from "../../api";

interface Errors {
  name?: string[];
  description?: string[];
  id_kategori?: string[];
  price?: string[];
  stock?: string[];
}

const name = ref("");
const description = ref("");
const id_kategori = ref<number | string>("");
const price = ref<number | string>("");
const stock = ref<number | string>("");

const isLoading = ref(true);
const errors = ref<Errors>({});

const route = useRoute();
const router = useRouter();

const fetchDetailProduct = async () => {
  try {
    const response = await Api.get(`/api/products/${route.params.id}`);

    // Cek di Console Browser (F12) untuk melihat bentuk asli JSON dari Backend
    console.log("RAW RESPONSE API:", response);
    console.log("RESPONSE DATA:", response.data);

    // Ambil objek produk dari berbagai kemungkinan struktur wrapper API
    let product = response.data;
    
    // Jika API menggunakan Resource / Standard Wrapper Laravel
    if (product && product.data) {
      product = product.data;
    }
    // Antisipasi jika ada double wrapper (product.data.data)
    if (product && product.data) {
      product = product.data;
    }

    console.log("PRODUK SETELAH DI-EXTRACT:", product);

    if (product) {
      // Mapping field dengan beberapa alternatif nama kolom database
      name.value = product.name || product.title || product.nama_produk || "";
      description.value = product.description || product.deskripsi || "";
      id_kategori.value = product.id_kategori || product.kategori_id || product.category_id || "";
      price.value = product.price || product.harga || "";
      stock.value = product.stock || product.stok || "";
    }
  } catch (error) {
    console.error("Gagal mengambil detail produk:", error);
  } finally {
    isLoading.value = false;
  }
};

onMounted(() => {
  fetchDetailProduct();
});

const updateProduct = async () => {
  try {
    await Api.put(`/api/products/${route.params.id}`, {
      name: name.value,
      description: description.value,
      id_kategori: id_kategori.value,
      price: price.value,
      stock: stock.value,
    });

    router.push("/products");
  } catch (error: any) {
    errors.value = error.response?.data?.errors || error.response?.data || {};
  }
};
</script>

<template>
  <div class="container mt-4 mb-5">
    <div class="row">
      <div class="col-md-12">
        <!-- Header Panel -->
        <div class="card border-0 shadow-sm rounded-0 mb-3" style="border-left: 5px solid #0d6efd !important;">
          <div class="card-body bg-light py-3 d-flex justify-content-between align-items-center">
            <div>
              <h5 class="fw-bold mb-0 text-uppercase text-dark" style="letter-spacing: 0.5px;">
                <i class="bi bi-pencil-square me-2 text-primary"></i>Edit Data Produk
              </h5>
            </div>
            <router-link to="/products" class="btn btn-secondary btn-sm px-3 rounded-0 shadow-sm fw-semibold">
              <i class="bi bi-arrow-left me-1"></i> Kembali
            </router-link>
          </div>
        </div>

        <!-- Indicator Loading -->
        <div v-if="isLoading" class="card border shadow-sm rounded-0 p-5 text-center">
          <div class="spinner-border text-primary mx-auto mb-3" role="status">
            <span class="visually-hidden">Loading...</span>
          </div>
          <span class="text-muted small">Memuat data produk...</span>
        </div>

        <!-- Main Form Card -->
        <div v-else class="card border shadow-sm rounded-0">
          <div class="card-body p-4">
            <form @submit.prevent="updateProduct">
              <div class="row">
                <div class="col-md-8">
                  <!-- Nama Produk -->
                  <div class="mb-3">
                    <label class="form-label fw-bold small text-uppercase">Nama Produk</label>
                    <input
                      type="text"
                      v-model="name"
                      class="form-control rounded-0"
                      placeholder="Masukkan Nama Produk"
                    />
                    <div v-if="errors.name" class="text-danger small mt-1">
                      {{ errors.name[0] }}
                    </div>
                  </div>

                  <!-- Deskripsi -->
                  <div class="mb-3">
                    <label class="form-label fw-bold small text-uppercase">Deskripsi Produk</label>
                    <textarea
                      v-model="description"
                      class="form-control rounded-0"
                      rows="5"
                      placeholder="Masukkan Deskripsi Produk"
                    ></textarea>
                    <div v-if="errors.description" class="text-danger small mt-1">
                      {{ errors.description[0] }}
                    </div>
                  </div>
                </div>

                <div class="col-md-4">
                  <!-- ID Kategori -->
                  <div class="mb-3">
                    <label class="form-label fw-bold small text-uppercase">ID Kategori</label>
                    <input
                      type="number"
                      v-model.number="id_kategori"
                      class="form-control rounded-0"
                      placeholder="ID Kategori"
                    />
                    <div v-if="errors.id_kategori" class="text-danger small mt-1">
                      {{ errors.id_kategori[0] }}
                    </div>
                  </div>

                  <!-- Harga -->
                  <div class="mb-3">
                    <label class="form-label fw-bold small text-uppercase">Harga (Rp)</label>
                    <input
                      type="number"
                      v-model.number="price"
                      class="form-control rounded-0"
                      placeholder="Harga Produk"
                    />
                    <div v-if="errors.price" class="text-danger small mt-1">
                      {{ errors.price[0] }}
                    </div>
                  </div>

                  <!-- Stok -->
                  <div class="mb-3">
                    <label class="form-label fw-bold small text-uppercase">Stok Produk</label>
                    <input
                      type="number"
                      v-model.number="stock"
                      class="form-control rounded-0"
                      placeholder="Jumlah Stok"
                    />
                    <div v-if="errors.stock" class="text-danger small mt-1">
                      {{ errors.stock[0] }}
                    </div>
                  </div>
                </div>
              </div>

              <hr class="my-4" />

              <!-- Tombol Aksi -->
              <div class="d-flex justify-content-end gap-2">
                <router-link to="/products" class="btn btn-secondary rounded-0 px-4">
                  Batal
                </router-link>
                <button type="submit" class="btn btn-primary rounded-0 px-4">
                  <i class="bi bi-save me-1"></i> Simpan Perubahan
                </button>
              </div>
            </form>
          </div>

          <!-- Footer Panel -->
          <div class="card-footer bg-light border-top py-2 px-3 small text-muted d-flex justify-content-between">
            <span>ID Produk: <strong>#{{ route.params.id }}</strong></span>
            <span>Sistem Informasi Produk</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>