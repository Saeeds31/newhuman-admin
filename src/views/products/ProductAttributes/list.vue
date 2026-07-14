<template>
  <div class="container mt-4" v-if="checkPermission(['productattribute_view'])">
    <div class="card">
      <div class="card-header d-flex justify-content-between align-items-center">
        <h3><i class="bi bi-list-ul"></i> مدیریت ویژگی‌ها</h3>
        <button class="btn btn-success" @click="showCreateModal = true" v-if="checkPermission(['productattribute_store'])">
          <i class="bi bi-plus"></i> افزودن ویژگی
        </button>
      </div>
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary"></div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
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
                  <button class="btn btn-sm btn-warning me-2" @click="editItem(item)" v-if="checkPermission(['productattribute_update'])">
                    <i class="bi bi-pen"></i>
                  </button>
                  <button class="btn btn-sm btn-danger" @click="deleteItem(item.id)" v-if="checkPermission(['productattribute_delete'])">
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

    <!-- مودال ایجاد/ویرایش ویژگی -->
    <div class="modal" :class="{ 'd-block': showCreateModal || showEditModal }" tabindex="-1">
      <div class="modal-dialog">
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
</style>