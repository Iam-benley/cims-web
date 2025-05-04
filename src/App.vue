<template>
  <div class="container mt-5">
    <div class="row mb-3">
      <div class="col-md-8">
        <div class="input-group">
          <input
            v-model="searchQuery"
            type="text"
            class="form-control"
            placeholder="Search by name..."
          />
          <select v-model="selectedCategory" class="form-select">
            <option value="">All Categories</option>
            <option
              v-for="category in categories"
              :key="category"
              :value="category"
            >
              {{ category }}
            </option>
          </select>
          <button class="btn btn-outline-primary" type="button">
            <i class="bi bi-search"></i> Search
          </button>
        </div>
      </div>
      <div class="col-md-4 text-end">
        <button @click="showDialog = true" class="btn btn-success">
          <i class="bi bi-plus-lg"></i> Add Item
        </button>
      </div>
    </div>

    <table class="table table-striped table-bordered">
      <thead class="table-dark">
        <tr>
          <th>Name</th>
          <th>Category</th>
          <th>Quantity</th>
          <th>Expiry Date</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in items" :key="item.id">
          <td>{{ item.name }}</td>
          <td>{{ item.category }}</td>
          <td>{{ item.quantity }}</td>
          <td>{{ item.expiryDate }}</td>
          <td>
            <div class="d-flex gap-2">
              <button
                @click="editItem(item)"
                class="btn btn-sm btn-outline-primary"
                title="Edit"
              >
                Edit
              </button>
              <button
                @click="confirmDelete(item)"
                class="btn btn-sm btn-outline-danger"
                title="Delete"
              >
                Delete
              </button>
            </div>
          </td>
        </tr>
      </tbody>
    </table>

    <BootstrapDialog
      v-model:show="showDialog"
      title="Confirmation"
      message="Are you sure you want to proceed?"
      size="lg"
      @confirm="handleConfirm"
      @close="handleClose"
    >
      <!-- Custom body content -->
      <template #body>
        <form @submit.prevent="submitForm">
          <div class="mb-3">
            <label for="itemName" class="form-label"
              >Item Name <span class="text-danger">(required)</span></label
            >
            <input
              type="text"
              class="form-control"
              id="itemName"
              v-model="form.name"
              required
            />
          </div>

          <div class="mb-3">
            <label for="category" class="form-label">Category</label>
            <select
              class="form-select"
              id="category"
              v-model="form.category_id"
              required
            >
              <option value="" disabled selected>Select a category</option>
              <option value="Medicine">Medicine</option>
              <option value="Equipment">Equipment</option>
              <option value="Consumables">Consumables</option>
            </select>
          </div>

          <div class="row mb-3">
            <div class="col-md-6">
              <label for="quantity" class="form-label">Quantity</label>
              <input
                type="number"
                class="form-control"
                id="quantity"
                min="0"
                v-model.number="form.quantity"
                required
              />
            </div>
            <div class="col-md-6">
              <label for="unit" class="form-label">Unit</label>
              <select
                class="form-select"
                id="unit"
                v-model="form.unit_id"
                required
              >
                <option value="" disabled selected>Select unit</option>
                <option value="pcs">pcs</option>
                <option value="bottles">bottles</option>
                <option value="boxes">boxes</option>
              </select>
            </div>
          </div>

          <div class="mb-3">
            <label for="expiryDate" class="form-label"
              >Expiry Date (optional)</label
            >
            <input
              type="date"
              class="form-control"
              id="expiryDate"
              v-model="form.expiry_date"
            />
          </div>

          <div class="d-grid gap-2">
            <button type="submit" class="btn btn-primary">Submit</button>
          </div>
        </form>
      </template>

      <!-- Custom footer -->
      <template #footer>
        <!-- <button class="btn btn-outline-danger" @click="showDialog = false">
          Cancel
        </button>
        <button class="btn btn-success" @click="handleConfirm">Proceed</button> -->
      </template>
    </BootstrapDialog>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import BootstrapDialog from "./components/ModalDialog.vue";
import axios from "axios";

const showDialog = ref(false);
const items = ref([]);

const searchData = ref({
  name: "",
  category_id: "",
});

const form = ref({
  name: "",
  category_id: "",
  quantity: null,
  unit_id: "",
  expiry_date: "",
});

async function submitForm() {
  form.value.category_id = 1;
  form.value.unit_id = 1;

  try {
    const response = await axios.post("http://cims.test", {
      method: "addItem",
      data: form.value,
    });

    console.log(response);
  } catch (error) {}
}

async function getInventoryItems() {
  try {
    const response = await axios.post("http://cims.test", {
      method: "getInventory",
      data: searchData.value,
    });
    console.log(response);
    if (response.status === 200) {
      items.value = response.data.data;
    }
  } catch (error) {
    console.log(error);
  }
}

const handleConfirm = () => {
  console.log("User confirmed the action");
  // Perform your action here
};

const handleClose = () => {
  console.log("Dialog was closed");
};

onMounted(async () => {
  await getInventoryItems();
});
</script>
