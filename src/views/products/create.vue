<script setup lang="ts">
import { ref, onMounted } from "vue";
import { useRouter } from "vue-router";
import Api from "../../api";

interface Category {
  id: number;
  name: string;
}

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

const categories = ref<Category[]>([]);
const isLoadingCategories = ref(true);

const errors = ref<Errors>({});
const router = useRouter();

// Mengambil daftar kategori dari API
const fetchCategories = async () => {
  try {
    const response = await Api.get("/api/kategori");
    
    // Mengakomodasi berbagai format response Laravel (Resource/Paginated/Array)
    let categoryData = response.data;
    if (categoryData && categoryData.data) {
      categoryData = categoryData.data;
    }
    
    categories.value = Array.isArray(categoryData) ? categoryData : [];
  } catch (error) {
    console.error("Gagal mengambil data kategori:", error);
  } finally {
    isLoadingCategories.value = false;
  }
};

onMounted(() => {
  fetchCategories();
});

const storeProduct = async () => {
  try {
    await Api.post("/api/products", {
      name: name.value,
      description: description.value,
      id_kategori: id_kategori.value,
      price: price.value,
      stock: stock.value
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
                <i class="bi bi-plus-square me-2 text-primary"></i>Tambah Produk Baru
              </h5>
            </div>
            <router-link to="/products" class="btn btn-secondary btn-sm px-3 rounded-0 shadow-sm fw-semibold">
              <i class="bi bi-arrow-left me-1"></i> Kembali
            </router-link>
          </div>
        </div>

        <!-- Main Form Card -->
        <div class="card border shadow-sm rounded-0">
          <div class="card-body p-4">
            <form @submit.prevent="storeProduct">
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
                  <!-- Dropdown Kategori -->
                  <div class="mb-3">
                    <label class="form-label fw-bold small text-uppercase">Kategori Produk</label>
                    <select
                      v-model="id_kategori"
                      class="form-select rounded-0"
                      :disabled="isLoadingCategories"
                    >
                      <option value="" disabled>
                        {{ isLoadingCategories ? 'Memuat kategori...' : '-- Pilih Kategori --' }}
                      </option>
                      <option
                        v-for="category in categories"
                        :key="category.id"
                        :value="category.id"
                      >
                        {{ category.name || (category as any).nama_kategori }}
                      </option>
                    </select>
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
                  <i class="bi bi-save me-1"></i> Simpan Produk
                </button>
              </div>
            </form>
          </div>

          <!-- Footer Panel -->
          <div class="card-footer bg-light border-top py-2 px-3 small text-muted d-flex justify-content-between">
            <span>Form Input Data Produk</span>
            <span>Sistem Informasi Produk</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>