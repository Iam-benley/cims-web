<template>
  <div class="container">
    <!-- Navigation Bar with Logo and Title -->
    <nav class="navbar navbar-light bg-light mb-4 rounded">
      <div class="container-fluid">
        <div class="d-flex align-items-center">
          <!-- Circular Logo Placeholder -->
          <div class="logo-placeholder me-3">
            <span class="logo-text">Logo</span>
          </div>
          <!-- System Title -->
          <h1 class="h4 mb-0">Clinic Inventory Management System</h1>
        </div>
      </div>
    </nav>

    <div class="row mb-3">
      <!-- Search Input Group (Full-width on mobile, 2/3 width on desktop) -->
      <div class="col-12 col-md-8 mb-2 mb-md-0">
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

      <!-- Add Item Button (Full-width on mobile, auto-width on desktop) -->
      <div class="col-12 col-md-4">
        <button
          @click="showDialog = true"
          class="btn btn-success w-100 d-md-inline-block d-md-auto"
        >
          <i class="bi bi-plus-lg"></i> Add Item
        </button>
      </div>
    </div>

    <div class="table-responsive">
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
                    @click="openGenerateReportDialog(item)"
                    class="btn btn-sm btn-outline-primary"
                    title="Edit"
                  >
                    Generate Report
                  </button>
                  <!-- <button
                  @click="openDispenseDialog(item)"
                  class="btn btn-sm btn-outline-danger"
                  title="Edit"
                >
                  Dispense
                </button> -->
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
    </div>

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
              maxlength="50"
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

    <BootstrapDialog
      v-model:show="showDispenseDialog"
      title="Dispense Item"
      message="Are you sure you want to proceed?"
      size="lg"
      @confirm="handleConfirm"
      @close="handleClose"
    >
      <!-- Custom body content -->
      <template #body>
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
      </template>

      <!-- Custom footer -->
      <template #footer>
        <button
          class="btn btn-outline-primary"
          @click="showDispenseDialog = false"
        >
          Cancel
        </button>
        <button class="btn btn-danger" @click="handleDeleteItem">
          Dispense
        </button>
      </template>
    </BootstrapDialog>

    <BootstrapDialog
      v-model:show="showGenerateReportDialog"
      title="Generate Report"
      message="Are you sure you want to proceed?"
      size="lg"
      @confirm="handleConfirm"
      @close="handleClose"
    >
      <!-- Custom body content -->
      <template #body>
        <div class="mb-3">
          <label for="expiryMonthYear" class="form-label">Report Month </label>
          <input
            type="month"
            class="form-control"
            id="expiryMonthYear"
            v-model="generateReportData.date"
          />
        </div>
        <div v-if="generateReportData.isShow">
          <table class="table table-striped table-bordered">
            <thead class="table-dark">
              <tr>
                <th>Name</th>
                <th>Total Dispensed</th>
                <th>Date</th>
              </tr>
            </thead>
            <tbody>
              <tr :key="generateReportData.item.id">
                <td>{{ generateReportData.item.name }}</td>
                <td>{{ generateReportData.total_dispensed }}</td>
                <td>{{ generateReportData.date }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </template>

      <!-- Custom footer -->
      <template #footer>
        <button class="btn btn-danger" @click="showDispenseDialog = false">
          Cancel
        </button>
        <button class="btn btn-success" @click="handleGenerateReport">
          Generate
        </button>
      </template>
    </BootstrapDialog>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import BootstrapDialog from "./components/ModalDialog.vue";
import axios from "axios";
import Toast from "./services/toast";

const baseUrl = import.meta.env.VITE_BASE_URL;

const categories = ref([]);
const units = ref([]);

//dialogs
const showDialog = ref(false);
const showDeleteDialog = ref(false);
const showDispenseDialog = ref(false);
const showGenerateReportDialog = ref(false);

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
  expiry_date: null,
});

const generateReportData = ref({
  date: "",
  inventory_id: "",
  item: {},
  total_dispensed: 0,
  isShow: false,
});

async function submitForm() {
  // await Toast.fire({
  //   icon: "success",
  //   title: "Successfully",
  // });

  // Ensure empty dates are null
  const cleanForm = {
    ...form.value,
    expiry_date: form.value.expiry_date || null,
    quantity: form.value.quantity || 0, // Also good to handle potential null quantities
  };

  if (isEdit.value) {
    console.log(form.value);
    try {
      const response = await axios.post(baseUrl, {
        method: "updateItem",
        data: form.value,
      });

      console.log(response);

      if (response.status === 200) {
        await getInventoryItems();
        showDialog.value = false;
        clearForm();
        await Toast.fire({
          icon: "success",
          title: "Item successfully updated!",
        });
      }
    } catch (error) {
      console.log(error);
    }
  } else {
    try {
      const response = await axios.post(baseUrl, {
        method: "addItem",
        data: cleanForm,
      });
      console.log(response);

      if (response.status === 200) {
        await getInventoryItems();
        showDialog.value = false;
        clearForm();
        await Toast.fire({
          icon: "success",
          title: "Item successfully added!",
        });
      }
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

    if (response.status === 200) {
      await getInventoryItems();
      showDeleteDialog.value = false;
      await Toast.fire({
        icon: "success",
        title: "Item successfully deleted!",
      });
    }
  } catch (error) {
    console.log(error);
  }
}

async function handleGenerateReport() {
  try {
    const response = await axios.post(baseUrl, {
      method: "getDispenseReport",
      data: generateReportData.value,
    });

    console.log(response);

    if (response.status === 200) {
      if (response.data.data[0].total_dispensed) {
        generateReportData.value.total_dispensed =
          response.data.data[0].total_dispensed;
        generateReportData.value.isShow = true;
      }
    }
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

function openDispenseDialog(item) {
  showDispenseDialog.value = true;
}

function openGenerateReportDialog(item) {
  showGenerateReportDialog.value = true;
  generateReportData.value.inventory_id = item.id;
  generateReportData.value.item = item;
}

function editItem(item) {
  form.value = item;
  isEdit.value = true;
  showDialog.value = true;
}

function clearForm() {
  form.value = {
    name: "",
    category_id: "",
    quantity: null,
    unit_id: "",
    expiry_date: "",
  };
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

<style scoped>
.logo-placeholder {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background-color: #e9ecef;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 12px;
  color: #6c757d;
  font-weight: bold;
}

.navbar {
  padding: 0.75rem 1rem;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}
</style>
