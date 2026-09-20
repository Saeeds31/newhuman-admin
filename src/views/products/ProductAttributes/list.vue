<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['productattribute_view'])">
    <div class="card header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2">
          <h3 class="mb-0 page-title">
            <i class="bi bi-list-ul"></i>
            <span>مدیریت ویژگی‌ها</span>
          </h3>
          <button class="btn btn-success add-btn" @click="showCreateModal = true" v-if="checkPermission(['productattribute_store'])">
            <i class="bi bi-plus"></i>
            <span>افزودن ویژگی</span>
          </button>
        </div>
      </div>
      <div class="card-body p-2 p-md-3">
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
                  <th>نام ویژگی</th>
                  <th>slug</th>
                  <th>نوع محصول</th>
                  <th>اجباری</th>
                  <th>عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in items.data" :key="item.id">
                  <td>{{ item.id }}</td>
                  <td>{{ item.name }}</td>
                  <td>{{ item.slug }}</td>
                  <td>{{ item.product_type?.name }}</td>
                  <td>
                    <span :class="item.is_required ? 'badge bg-warning' : 'badge bg-secondary'">
                      {{ item.is_required ? 'اجباری' : 'اختیاری' }}
                    </span>
                  </td>
                  <td>
                    <div class="d-flex flex-wrap gap-1">
                      <button class="btn btn-sm btn-warning" @click="editItem(item)" v-if="checkPermission(['productattribute_update'])">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </button>
                      <button class="btn btn-sm btn-danger" @click="deleteItem(item.id)" v-if="checkPermission(['productattribute_delete'])">
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
          <div class="d-md-none attribute-cards">
            <div
              v-for="item in items.data"
              :key="item.id"
              class="attribute-card"
            >
              <div class="attribute-card-header">
                <div class="attribute-id-badge">#{{ item.id }}</div>
                <div class="attribute-name">{{ item.name }}</div>
                <span :class="item.is_required ? 'badge bg-warning' : 'badge bg-secondary'">
                  {{ item.is_required ? 'اجباری' : 'اختیاری' }}
                </span>
              </div>

              <div class="attribute-card-body">
                <div class="attribute-info-row">
                  <i class="bi bi-link-45deg"></i>
                  <span class="info-label">slug:</span>
                  <span class="info-value">{{ item.slug }}</span>
                </div>
                <div class="attribute-info-row">
                  <i class="bi bi-tag"></i>
                  <span class="info-label">نوع محصول:</span>
                  <span class="info-value">{{ item.product_type?.name ?? '-' }}</span>
                </div>
              </div>

              <div class="attribute-card-actions">
                <button class="btn btn-sm btn-warning flex-fill" @click="editItem(item)" v-if="checkPermission(['productattribute_update'])">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </button>
                <button class="btn btn-sm btn-danger flex-fill" @click="deleteItem(item.id)" v-if="checkPermission(['productattribute_delete'])">
                  <i class="bi bi-trash3"></i>
                  <span>حذف</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!items.data || items.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>ویژگی‌ای یافت نشد</p>
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

    <!-- مودال ایجاد/ویرایش ویژگی -->
    <div class="modal" :class="{ 'd-block': showCreateModal || showEditModal }" tabindex="-1">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">{{ showEditModal ? 'ویرایش ویژگی' : 'ایجاد ویژگی جدید' }}</h5>
            <button type="button" class="btn-close" @click="closeModal"></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="submitAttribute">
              <div class="mb-3">
                <label class="form-label">نوع محصول</label>
                <select v-model="form.product_type_id" class="form-control" required>
                  <option value="">انتخاب کنید</option>
                  <option v-for="type in productTypes" :key="type.id" :value="type.id">
                    {{ type.name }}
                  </option>
                </select>
                <span v-if="errors.product_type_id" class="text-danger">{{ errors.product_type_id[0] }}</span>
              </div>

              <div class="mb-3">
                <label class="form-label">نام ویژگی</label>
                <input v-model="form.name" type="text" class="form-control" required />
                <span v-if="errors.name" class="text-danger">{{ errors.name[0] }}</span>
              </div>

              <div class="mb-3">
                <label class="form-label">slug</label>
                <input v-model="form.slug" type="text" class="form-control" required />
                <span v-if="errors.slug" class="text-danger">{{ errors.slug[0] }}</span>
              </div>

              <div class="mb-3">
                <div class="form-check">
                  <input v-model="form.is_required" type="checkbox" class="form-check-input" id="is_required" />
                  <label class="form-check-label" for="is_required">اجباری</label>
                </div>
              </div>

              <div class="mb-3">
                <label class="form-label">ترتیب</label>
                <input v-model.number="form.sort_order" type="number" class="form-control" min="0" />
              </div>

              <button :disabled="loading" type="submit" class="btn btn-primary">
                {{ showEditModal ? 'بروزرسانی' : 'ذخیره' }}
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>
    <div class="modal-backdrop" v-if="showCreateModal || showEditModal"></div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import axios from 'axios';
import Swal from 'sweetalert2';
import { toast } from 'vue3-toastify';
import { useRouter } from 'vue-router';
import { useAdmin } from '@/stores/modules/admin';

const store = useAdmin();
const checkPermission = store.checkPermission;
const router = useRouter();
const loading = ref(false);
const items = ref({ data: [], total: 0, last_page: 1, per_page: 20 });
const currentPage = ref(1);
const productTypes = ref([]);
const showCreateModal = ref(false);
const showEditModal = ref(false);
const editId = ref(null);
const errors = ref({});

const form = ref({
  product_type_id: '',
  name: '',
  slug: '',
  is_required: false,
  sort_order: 0
});

async function getItems(url = '/product-attributes') {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
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

function changePage(selectedPage) {
  getItems(`/products/product-attributes?page=${selectedPage}`);
}

function goToValues(id) {
  router.push(`/products/product-attributes/${id}/values`);
}

function editItem(item) {
  editId.value = item.id;
  form.value = { ...item };
  showEditModal.value = true;
}

function closeModal() {
  showCreateModal.value = false;
  showEditModal.value = false;
  form.value = { product_type_id: '', name: '', slug: '', is_required: false, sort_order: 0 };
  errors.value = {};
  editId.value = null;
}

async function submitAttribute() {
  errors.value = {};
  loading.value = true;
  try {
    if (showEditModal.value) {
      await axios.put(`/product-attributes/${editId.value}`, form.value);
      toast.success('ویژگی بروزرسانی شد');
    } else {
      await axios.post('/product-attributes', form.value);
      toast.success('ویژگی ایجاد شد');
    }
    closeModal();
    getItems();
  } catch (e) {
    if (e.response?.data?.errors) {
      errors.value = e.response.data.errors;
    }
    toast.error('خطا در ذخیره');
  } finally {
    loading.value = false;
  }
}

const deleteItem = (id) => {
  Swal.fire({
    title: 'حذف ویژگی',
    text: 'آیا مطمئن هستید؟',
    icon: 'warning',
    showCancelButton: true,
    confirmButtonText: 'بله، حذف شود',
    cancelButtonText: 'انصراف',
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/product-attributes/${id}`);
        Swal.fire('موفق', 'ویژگی حذف شد', 'success');
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
.attribute-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.attribute-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.attribute-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.attribute-card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.attribute-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.attribute-name {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
  min-width: 0;
}

.attribute-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.attribute-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.attribute-info-row i {
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
  direction: ltr;
}

.attribute-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.attribute-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.attribute-card-actions .btn span {
  display: none;
}

/* ===== Pagination ===== */
.pagination-responsive {
  flex-wrap: wrap;
  justify-content: center;
}

/* ===== Modal ===== */
.modal {
  background: rgba(0,0,0,0.5);
}
.modal-backdrop {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  z-index: 1040;
}
.modal {
  z-index: 1050;
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

  /* نمایش label دکمه‌ها در موبایل */
  .attribute-card-actions .btn span {
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

  .attribute-card {
    padding: 12px;
  }

  .attribute-name {
    font-size: 0.9rem;
  }

  .attribute-info-row {
    font-size: 0.78rem;
  }

  .attribute-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .attribute-cards {
    display: none;
  }
}
</style>