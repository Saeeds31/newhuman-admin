<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['product_view'])">
    <div class="card header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2">
          <h3 class="mb-0 page-title">
            <i class="bi bi-box"></i>
            <span>مدیریت محصولات</span>
          </h3>
          <router-link to="/products/create" class="btn btn-success add-btn" v-if="checkPermission(['product_store'])">
            <i class="bi bi-plus"></i>
            <span>افزودن محصول</span>
          </router-link>
        </div>
      </div>
      <div class="card-body p-2 p-md-3">
        <!-- فیلترها -->
        <div class="row g-2 mb-3">
          <div class="col-12 col-sm-6 col-md-3">
            <input v-model="filters.search" @input="applyFilters" class="form-control search-input" placeholder="جستجو..." />
          </div>
          <div class="col-12 col-sm-6 col-md-3">
            <select v-model="filters.product_type_id" @change="applyFilters" class="form-control">
              <option value="">همه انواع</option>
              <option v-for="type in productTypes" :key="type.id" :value="type.id">{{ type.name }}</option>
            </select>
          </div>
          <div class="col-12 col-sm-6 col-md-3">
            <select v-model="filters.status" @change="applyFilters" class="form-control">
              <option value="">همه وضعیت‌ها</option>
              <option value="draft">پیش‌نویس</option>
              <option value="published">منتشر شده</option>
              <option value="unpublished">منتشر نشده</option>
            </select>
          </div>
          <div class="col-12 col-sm-6 col-md-3">
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
          <!-- ===== نمایش جدول در دسکتاپ ===== -->
          <div class="table-responsive d-none d-md-block">
            <table class="table table-bordered table-striped mb-0">
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
                    <div class="d-flex flex-wrap gap-1">
                      <router-link :to="`/products/${item.id}/edit`" class="btn btn-sm btn-warning" v-if="checkPermission(['product_update'])">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button class="btn btn-sm btn-info" @click="toggleStatus(item.id)" v-if="checkPermission(['product_update'])">
                        <i class="bi bi-arrow-repeat"></i>
                        <span>وضعیت</span>
                      </button>
                      <button class="btn btn-sm btn-danger" @click="deleteItem(item.id)" v-if="checkPermission(['product_delete'])">
                        <i class="bi bi-trash3"></i>
                        <span>حذف</span>
                      </button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- ===== نمایش کارتی در موبایل ===== -->
          <div class="d-md-none product-cards">
            <div
              v-for="item in items.data"
              :key="item.id"
              class="product-card"
            >
              <div class="product-card-header">
                <div class="product-id-badge">#{{ item.id }}</div>
                <div class="product-title">{{ item.title }}</div>
              </div>

              <div class="product-card-image" v-if="item.main_image">
                <img :src="baseImageAddress + item.main_image" alt="" class="product-image">
              </div>

              <div class="product-card-body">
                <div class="product-info-row">
                  <i class="bi bi-tag"></i>
                  <span class="info-label">نوع:</span>
                  <span class="info-value">{{ item.product_type?.name ?? '-' }}</span>
                </div>
                <div class="product-info-row">
                  <i class="bi bi-cash-stack"></i>
                  <span class="info-label">قیمت:</span>
                  <span class="info-value">
                    <span v-if="item.is_free" class="badge bg-success">رایگان</span>
                    <span v-else>{{ numberFormat(item.final_price) }} تومان</span>
                  </span>
                </div>
                <div class="product-info-row">
                  <i class="bi bi-toggle-on"></i>
                  <span class="info-label">وضعیت:</span>
                  <span :class="{
                    'badge bg-secondary': item.status === 'draft',
                    'badge bg-success': item.status === 'published',
                    'badge bg-danger': item.status === 'unpublished'
                  }">
                    {{ item.status === 'draft' ? 'پیش‌نویس' : item.status === 'published' ? 'منتشر شده' : 'منتشر نشده' }}
                  </span>
                </div>
              </div>

              <div class="product-card-actions">
                <router-link :to="`/products/${item.id}/edit`" class="btn btn-sm btn-warning flex-fill" v-if="checkPermission(['product_update'])">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button class="btn btn-sm btn-info flex-fill" @click="toggleStatus(item.id)" v-if="checkPermission(['product_update'])">
                  <i class="bi bi-arrow-repeat"></i>
                  <span>وضعیت</span>
                </button>
                <button class="btn btn-sm btn-danger flex-fill" @click="deleteItem(item.id)" v-if="checkPermission(['product_delete'])">
                  <i class="bi bi-trash3"></i>
                  <span>حذف</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!items.data || items.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>محصولی یافت نشد</p>
            </div>
          </div>

          <!-- Pagination -->
          <b-pagination
            v-model="currentPage"
            :total-rows="items.total"
            v-if="items.last_page != 1"
            :per-page="items.per_page"
            @Update:modelValue="changePage"
            align="center"
            class="mt-3 pagination-responsive">
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

<style scoped>
/* ===== هدر صفحه ===== */
.header-card .card-header {
  padding: 16px 20px;
  background: transparent;
  border-bottom: 2px solid #f8f9fa;
}

.page-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1.5rem;
  display: flex;
  align-items: center;
  gap: 8px;
}

.add-btn {
  white-space: nowrap;
  display: inline-flex;
  align-items: center;
  gap: 6px;
  justify-content: center;
}

.search-input {
  border-radius: 10px;
  padding: 10px 14px;
  border: 1px solid #e0e0e0;
  transition: all 0.2s ease;
}

.search-input:focus {
  border-color: #6c5ce7;
  box-shadow: 0 0 0 3px rgba(108, 92, 231, 0.1);
}

/* ===== جدول ===== */
.table {
  margin-bottom: 0;
}

.table thead th {
  background: #f8f9fa;
  font-weight: 600;
  color: #2d3436;
  white-space: nowrap;
  font-size: 0.9rem;
}

.table tbody td {
  vertical-align: middle;
  font-size: 0.9rem;
}

/* ===== کارت‌های موبایل ===== */
.product-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.product-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.product-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.product-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.product-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.product-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.product-card-image {
  display: flex;
  justify-content: center;
  margin-bottom: 12px;
}

.product-image {
  max-width: 100%;
  max-height: 150px;
  border-radius: 8px;
  border: 1px solid #e9ecef;
  object-fit: cover;
}

.product-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.product-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.product-info-row i {
  color: #6c5ce7;
  font-size: 0.95rem;
  width: 18px;
  text-align: center;
  flex-shrink: 0;
}

.info-label {
  color: #6c757d;
  flex-shrink: 0;
}

.info-value {
  color: #2d3436;
  font-weight: 600;
  margin-right: auto;
  word-break: break-word;
  text-align: left;
}

.product-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.product-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.product-card-actions .btn span {
  display: none;
}

/* ===== Pagination ===== */
.pagination-responsive {
  flex-wrap: wrap;
  justify-content: center;
}

/* ========================================= */
/* ===== موبایل (کمتر از 768px) ===== */
/* ========================================= */
@media (max-width: 767.98px) {
  .header-card .card-header {
    padding: 12px 14px;
  }

  .header-card .card-body {
    padding: 12px 14px;
  }

  .page-title {
    font-size: 1.15rem;
    justify-content: center;
    text-align: center;
    width: 100%;
  }

  .add-btn {
    width: 100%;
  }

  .search-input {
    padding: 9px 12px;
    font-size: 0.9rem;
  }

  /* نمایش label دکمه‌ها در موبایل */
  .product-card-actions .btn span {
    display: inline;
  }
}

/* ========================================= */
/* ===== موبایل کوچک (کمتر از 400px) ===== */
/* ========================================= */
@media (max-width: 399.98px) {
  .page-title {
    font-size: 1rem;
  }

  .product-card {
    padding: 12px;
  }

  .product-title {
    font-size: 0.9rem;
  }

  .product-info-row {
    font-size: 0.78rem;
  }

  .product-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }

  .product-image {
    max-height: 120px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .product-cards {
    display: none;
  }
}
</style>