<template>
  <div>

    <!-- NAVBAR (from your template) -->
    <nav class="navbar navbar-expand-lg navbar-light bg-white shadow-sm px-4">
      <a class="navbar-brand" href="#">Furnish</a>

      <div class="ms-auto">
        <button class="btn btn-dark" @click="openForm">
          + Add Product
        </button>
      </div>
    </nav>

    <!-- PRODUCTS SECTION -->
    <div class="container mt-4">

      <h2 class="mb-3">Products</h2>

      <!-- EMPTY STATE -->
      <div v-if="products.length === 0" class="alert alert-info">
        No products available 😢
      </div>

      <!-- PRODUCT CARDS -->
      <div class="row">
        <div class="col-md-4 mb-3" v-for="p in products" :key="p.id">

          <div class="card shadow-sm">
            <div class="card-body">
              <h5>{{ p.name }}</h5>
              <p>${{ p.price }}</p>

              <button class="btn btn-sm btn-warning me-2" @click="editProduct(p)">
                Edit
              </button>

              <button class="btn btn-sm btn-danger" @click="deleteProduct(p.id)">
                Delete
              </button>
            </div>
          </div>

        </div>
      </div>

    </div>

    <!-- MODAL FORM -->
    <div v-if="showForm" class="modal-backdrop-custom">
      <div class="modal-box">

        <h4>{{ editMode ? "Edit Product" : "Add Product" }}</h4>

        <input v-model="form.name" class="form-control mb-2" placeholder="Name" />
        <input v-model.number="form.price" class="form-control mb-2" type="number" placeholder="Price" />

        <p v-if="error" class="text-danger">{{ error }}</p>

        <button class="btn btn-success me-2" @click="saveProduct">
          Save
        </button>

        <button class="btn btn-secondary" @click="closeForm">
          Cancel
        </button>

      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";

const STORAGE_KEY = "products_cache";

const products = ref([]);
const showForm = ref(false);
const editMode = ref(false);
const error = ref("");

const form = ref({
  id: null,
  name: "",
  price: null,
});

// LOAD DATA (CACHE FIRST)
onMounted(() => {
  const cache = localStorage.getItem(STORAGE_KEY);

  if (cache) {
    products.value = JSON.parse(cache);
  } else {
    products.value = [
      { id: 1, name: "Laptop", price: 1200 },
      { id: 2, name: "Phone", price: 800 },
    ];
    updateCache();
  }
});

// OPEN FORM
const openForm = () => {
  resetForm();
  showForm.value = true;
};

// CLOSE FORM
const closeForm = () => {
  showForm.value = false;
  resetForm();
};

// RESET
const resetForm = () => {
  form.value = { id: null, name: "", price: null };
  editMode.value = false;
  error.value = "";
};

// CREATE / UPDATE
const saveProduct = () => {
  if (!form.value.name || form.value.price <= 0) {
    error.value = "Enter valid name and price";
    return;
  }

  if (editMode.value) {
    products.value = products.value.map(p =>
      p.id === form.value.id ? form.value : p
    );
  } else {
    products.value.push({
      id: Date.now(),
      name: form.value.name,
      price: form.value.price,
    });
  }

  updateCache();
  closeForm();
};

// EDIT
const editProduct = (p) => {
  form.value = { ...p };
  editMode.value = true;
  showForm.value = true;
};

// DELETE
const deleteProduct = (id) => {
  products.value = products.value.filter(p => p.id !== id);
  updateCache();
};

// CACHE UPDATE
const updateCache = () => {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(products.value));
};
</script>

<style>
.modal-backdrop-custom {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-box {
  background: white;
  padding: 20px;
  width: 400px;
  border-radius: 10px;
}
</style>