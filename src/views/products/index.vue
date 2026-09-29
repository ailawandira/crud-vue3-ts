<script setup lang="ts">
import { ref, onMounted } from "vue";
import Api from "../../api";

interface Kategori {
    id: number;
    name: string;
}

interface Product {
    id: number;
    name?: string;
    title?: string;
    description: string;
    price: number;
    stock: number;
    id_kategori?: number;
    kategori?: Kategori;
}

const products = ref<Product[]>([]);

const fetchDataProducts = async () => {
    try {
        const response = await Api.get("/api/products");
        products.value = response.data?.data?.data || response.data?.data || response.data;
    } catch (error) {
        console.error("Error fetching products:", error);
    }
};

const deleteProduct = async (id: number) => {
    if (confirm("Apakah Anda yakin ingin menghapus produk ini?")) {
        try {
            await Api.delete(`/api/products/${id}`);
            fetchDataProducts();
        } catch (error) {
            console.error("Error deleting product:", error);
        }
    }
};

onMounted(() => {
    fetchDataProducts();
});
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
                                <i class="bi bi-box-seam me-2 text-primary"></i>Manajemen Data Produk
                            </h5>
                        </div>
                        <router-link to="/products/create" class="btn btn-primary btn-sm px-3 rounded-0 shadow-sm fw-semibold">
                            <i class="bi bi-plus-lg me-1"></i> Tambah Produk
                        </router-link>
                    </div>
                </div>

                <!-- Main Content Card -->
                <div class="card border shadow-sm rounded-0">
                    <div class="card-body p-3">
                        <div class="table-responsive">
                            <table class="table table-hover table-striped table-bordered align-middle mb-0">
                                <thead class="table-dark text-center text-uppercase small">
                                    <tr>
                                        <th scope="col" style="width: 50px;">No</th>
                                        <th scope="col">Nama Produk</th>
                                        <th scope="col">Deskripsi</th>
                                        <th scope="col" style="width: 140px;">Kategori</th>
                                        <th scope="col" style="width: 150px;">Harga</th>
                                        <th scope="col" style="width: 90px;">Stok</th>
                                        <th scope="col" style="width: 160px;">Aksi</th>
                                    </tr>
                                </thead>
                                <tbody>
                                    <tr v-if="products.length === 0">
                                        <td colspan="7" class="text-center py-4 text-muted">
                                            <i class="bi bi-info-circle me-1"></i> Belum ada data produk.
                                        </td>
                                    </tr>
                                    <tr v-for="(product, index) in products" :key="product.id">
                                        <td class="text-center fw-bold text-secondary">{{ index + 1 }}</td>
                                        <td>
                                            <span class="fw-bold text-dark d-block">{{ product.name || product.title }}</span>
                                            <small class="text-muted">ID: #{{ product.id }}</small>
                                        </td>
                                        <td>{{ product.description }}</td>
                                        <td class="text-center">
                                            <span class="badge bg-secondary rounded-0 px-2 py-1">
                                                {{ product.kategori?.name || product.id_kategori || 'Umum' }}
                                            </span>
                                        </td>
                                        <td class="fw-bold text-dark text-end">
                                            Rp {{ product.price?.toLocaleString('id-ID') }}
                                        </td>
                                        <td class="text-center">
                                            <span :class="['badge rounded-0', product.stock > 0 ? 'bg-success' : 'bg-danger']">
                                                {{ product.stock }}
                                            </span>
                                        </td>
                                        <td class="text-center">
                                            <router-link :to="`/products/edit/${product.id}`"
                                                class="btn btn-warning btn-sm text-white rounded-0 me-1"
                                                title="Edit">
                                                <i class="bi bi-pencil-square"></i> Edit
                                            </router-link>
                                            <button @click="deleteProduct(product.id)" 
                                                class="btn btn-danger btn-sm rounded-0"
                                                title="Hapus">
                                                <i class="bi bi-trash"></i> Hapus
                                            </button>
                                        </td>
                                    </tr>
                                </tbody>
                            </table>
                        </div>
                    </div>
                    <!-- Footer Panel -->
                    <div class="card-footer bg-light border-top py-2 px-3 small text-muted d-flex justify-content-between">
                        <span>Total Data: <strong>{{ products.length }}</strong></span>
                        <span>Sistem Informasi Produk</span>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>