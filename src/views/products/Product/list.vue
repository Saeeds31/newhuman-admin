<template>
  <div class="container mt-4" v-if="checkPermission(['product_view'])">
    <div class="card">
      <div class="card-header d-flex justify-content-between align-items-center">
        <h3><i class="bi bi-box"></i> مدیریت محصولات</h3>
        <router-link to="/products/create" class="btn btn-success" v-if="checkPermission(['product_store'])">
          <i class="bi bi-plus"></i> افزودن محصول
        </router-link>
      </div>
      <div class="card-body">
        <!-- فیلترها -->
        <div class="row mb-3">
          <div class="col-md-3">
            <input v-model="filters.search" @input="applyFilters" class="form-control" placeholder="جستجو..." />
          </div>
          <div class="col-md-3">
            <select v-model="filters.product_type_id" @change="applyFilters" class="form-control">
              <option value="">همه انواع</option>
              <option v-for="type in productTypes" :key="type.id" :value="type.id">{{ type.name }}</option>
            </select>
          </div>
          <div class="col-md-3">
            <select v-model="filters.status" @change="applyFilters" class="form-control">
              <option value="">همه وضعیت‌ها</option>
              <option value="draft">پیش‌نویس</option>
              <option value="published">منتشر شده</option>
              <option value="unpublished">منتشر نشده</option>
            </select>
          </div>
          <div class="col-md-3">
            <select v-model="filters.is_free" @change="applyFilters" class="form-control">
              <option value="">همه</option>
              <option value="1">رایگان</option>
              <option value="0">پولی</option>
            </select>
          </div>
        </div>

        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary"></div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>تصویر</th>
                <th>عنوان</th>
                <th>نوع</th>
                <th>قیمت</th>
                <th>وضعیت</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in items.data" :key="item.id">
                <td>{{ item.id }}</td>
                <td>
                  <img v-if="item.main_image" :src="baseImageAddress + item.main_image" width="50" height="50" class="rounded" />
                  <span v-else>-</span>
                </td>
                <td>{{ item.title }}</td>
                <td>{{ item.product_type?.name }}</td>
                <td>
                  <span v-if="item.is_free" class="badge bg-success">رایگان</span>
                  <span v-else>{{ numberFormat(item.final_price) }} تومان</span>
                </td>
                <td>
                  <span :class="{
                    'badge bg-secondary': item.status === 'draft',
                    'badge bg-success': item.status === 'published',
                    'badge bg-danger': item.status === 'unpublished'
                  }">
                    {{ item.status === 'draft' ? 'پیش‌نویس' : item.status === 'published' ? 'منتشر شده' : 'منتشر نشده' }}
                  </span>
                </td>
                <td>
                  <router-link :to="`/products/${item.id}/edit`" class="btn btn-sm btn-warning me-2" v-if="checkPermission(['product_update'])">
                    <i class="bi bi-pen"></i>
                  </router-link>
                  <button class="btn btn-sm btn-info me-2" @click="toggleStatus(item.id)" v-if="checkPermission(['product_update'])">
                    <i class="bi bi-arrow-repeat"></i>
                  </button>
                  <button class="btn btn-sm btn-danger" @click="deleteItem(item.id)" v-if="checkPermission(['product_delete'])">
                    <i class="bi bi-trash3"></i>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>

          <b-pagination
            v-model="currentPage"
            :total-rows="items.total"
            v-if="items.last_page != 1"
            :per-page="items.per_page"
            @Update:modelValue="changePage"
            align="center"
            class="mt-3">
          </b-pagination>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import Swal from 'sweetalert2';
import { toast } from 'vue3-toastify';
import { useAdmin } from '@/stores/modules/admin';

const store = useAdmin();
const checkPermission = store.checkPermission;
const baseImageAddress = window.baseImageAddress;

const loading = ref(false);
const items = ref({ data: [], total: 0, last_page: 1, per_page: 20 });
const currentPage = ref(1);
const productTypes = ref([]);
const filters = ref({ search: '', product_type_id: '', status: '', is_free: '' });

async function getItems(url = '/products') {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    items.value = data.data;
    currentPage.value = data.data.current_page;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}

async function loadProductTypes() {
  try {
    const { data } = await axios.get('/product-types');
    productTypes.value = data.data.data;
  } catch (err) {
    console.error(err);
  }
}

function applyFilters() {
  getItems('/products');
}

function changePage(selectedPage) {
  getItems(`/products?page=${selectedPage}`);
}

function toggleStatus(id) {
  axios.patch(`/products/${id}/toggle-status`)
    .then(() => {
      toast.success('وضعیت تغییر کرد');
      getItems();
    })
    .catch(() => toast.error('خطا در تغییر وضعیت'));
}

function numberFormat(num) {
  return new Intl.NumberFormat().format(num);
}

const deleteItem = (id) => {
  Swal.fire({
    title: 'حذف محصول',
    text: 'آیا مطمئن هستید؟',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonText: 'بله، حذف شود',
    cancelButtonText: 'انصراف',
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/products/${id}`);
        Swal.fire('موفق', 'محصول حذف شد', 'success');
        getItems();
      } catch (err) {
        Swal.fire('خطا', 'مشکلی در حذف پیش آمد', 'error');
      }
    }
  });
};

onMounted(() => {
  getItems();
  loadProductTypes();
});
</script>