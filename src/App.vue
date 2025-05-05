<template>
  <div class="container mt-5">
    <div class="row mb-3">
      <div class="col-md-8">
        <div class="input-group">
          <input
            v-model="searchData.name"
            type="text"
            class="form-control"
            placeholder="Search by name..."
          />
          <select v-model="searchData.category_id" class="form-select">
            <option value="">All Categories</option>
            <option
              v-for="category in categories"
              :key="category.id"
              :value="category.id"
            >
              {{ category.name }}
            </option>
          </select>
          <button
            @click="getInventoryItems"
            class="btn btn-outline-primary"
            type="button"
          >
            Search
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
        <template v-if="items.length > 0">
          <tr v-for="item in items" :key="item.id">
            <td>{{ item.name }}</td>
            <td>{{ item.category_name }}</td>
            <td>{{ item.quantity }}</td>
            <td>{{ item.expiry_date ? item.expiry_date : "N/A" }}</td>
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
                  @click="openDeleteDialog(item)"
                  class="btn btn-sm btn-outline-danger"
                  title="Delete"
                >
                  Delete
                </button>
              </div>
            </td>
          </tr>
        </template>
        <template v-else>
          <tr>
            <td colspan="5" class="text-center text-muted py-4">
              No data available
            </td>
          </tr>
        </template>
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
              <option
                v-for="category in categories"
                :key="category.id"
                :value="category.id"
              >
                {{ category.name }}
              </option>
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
                <option v-for="unit in units" :key="unit.id" :value="unit.id">
                  {{ unit.abbreviation }}
                </option>
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
            <button type="submit" class="btn btn-primary">
              {{ isEdit ? "Update" : "Submit" }}
            </button>
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

    <BootstrapDialog
      v-model:show="showDeleteDialog"
      title="Confirmation"
      message="Are you sure you want to proceed?"
      size="lg"
      @confirm="handleConfirm"
      @close="handleClose"
    >
      <!-- Custom body content -->
      <template #body>
        <h4>Are you sure?</h4>
        <p>You won't be able to revert this!</p>
      </template>

      <!-- Custom footer -->
      <template #footer>
        <button
          class="btn btn-outline-primary"
          @click="showDeleteDialog = false"
        >
          Cancel
        </button>
        <button class="btn btn-danger" @click="handleDeleteItem">Delete</button>
      </template>
    </BootstrapDialog>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import BootstrapDialog from "./components/ModalDialog.vue";
import axios from "axios";

const baseUrl = import.meta.env.VITE_BASE_URL;

const categories = ref([]);
const units = ref([]);

const showDialog = ref(false);
const showDeleteDialog = ref(false);
const isEdit = ref(false);
const items = ref([]);
const itemToDelete = ref({});

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
  if (isEdit.value) {
    console.log(form.value);
    try {
      const response = await axios.post(baseUrl, {
        method: "updateItem",
        data: form.value,
      });

      console.log(response);
    } catch (error) {
      console.log(error);
    }
  } else {
    console.log("add item");
    console.log(form.value);

    try {
      const response = await axios.post(baseUrl, {
        method: "addItem",
        data: form.value,
      });
      console.log(response);
    } catch (error) {
      console.log(error);
    }
  }
}

async function getInventoryItems() {
  try {
    const response = await axios.post(baseUrl, {
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

async function handleDeleteItem() {
  try {
    console.log(itemToDelete.value);

    const response = await axios.post(baseUrl, {
      method: "deleteItem",
      data: itemToDelete.value,
    });

    console.log(response);
  } catch (error) {
    console.log(error);
  }
}

async function getCategories() {
  try {
    const response = await axios.post(baseUrl, {
      method: "getCategories",
      data: "",
    });

    console.log(response);

    if (response.status == 200) {
      categories.value = response.data.data;
    }
  } catch (error) {
    console.log(error);
  }
}

async function getUnits() {
  try {
    const response = await axios.post(baseUrl, {
      method: "getUnits",
      data: "",
    });

    console.log(response);

    if (response.status == 200) {
      units.value = response.data.data;
    }
  } catch (error) {
    console.log(error);
  }
}

function openDeleteDialog(item) {
  showDeleteDialog.value = true;
  itemToDelete.value = item;
}

function editItem(item) {
  form.value = item;
  isEdit.value = true;
  showDialog.value = true;
}

function handleConfirm() {
  console.log("User confirmed the action");
  // Perform your action here
}

function handleClose() {
  console.log("Dialog was closed");
}

onMounted(async () => {
  await getInventoryItems();
  await getCategories();
  await getUnits();
});
</script>
