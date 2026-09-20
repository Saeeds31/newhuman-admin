<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['producttype_view'])">
    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-tags"></i>
            <span>مدیریت  نوع محصول</span>
          </h3>
          <router-link v-if="checkPermission(['producttype_store'])" to="/products/product-types/create"
            class="btn btn-success add-btn">
            <i class="bi bi-plus"></i>
            <span>افزودن دسته‌بندی</span>
          </router-link>
        </div>
      </div>
    </div>

    <div class="card">
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
                  <th>نام</th>
                  <th>slug</th>
                  <th>وضعیت</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="item in items.data" :key="item.id">
                  <td>{{ item.id }}</td>
                  <td>{{ item.name }}</td>
                  <td>{{ item.slug }}</td>
                  <td>
                    <span :class="item.is_active ? 'badge bg-success' : 'badge bg-danger'">
                      {{ item.is_active ? 'فعال' : 'غیرفعال' }}
                    </span>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- ===== نمایش کارتی در موبایل ===== -->
          <div class="d-md-none type-cards">
            <div
              v-for="item in items.data"
              :key="item.id"
              class="type-card"
            >
              <div class="type-card-header">
                <div class="type-id-badge">#{{ item.id }}</div>
                <div class="type-name">{{ item.name }}</div>
              </div>

              <div class="type-card-body">
                <div class="type-info-row">
                  <i class="bi bi-link-45deg"></i>
                  <span class="info-label">slug:</span>
                  <span class="info-value">{{ item.slug }}</span>
                </div>
                <div class="type-info-row">
                  <i class="bi bi-toggle-on"></i>
                  <span class="info-label">وضعیت:</span>
                  <span :class="item.is_active ? 'badge bg-success' : 'badge bg-danger'">
                    {{ item.is_active ? 'فعال' : 'غیرفعال' }}
                  </span>
                </div>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!items.data || items.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>دسته‌بندی‌ای یافت نشد</p>
            </div>
          </div>

          <!-- Pagination -->
          <b-pagination v-model="currentPage" :total-rows="items.total" v-if="items.last_page != 1"
            :per-page="items.per_page" @Update:modelValue="changePage" align="center"
            class="mt-3 pagination-responsive">
          </b-pagination>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useRoute, useRouter } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';

const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute();
const router = useRouter();
const currentPage = ref(1);
const items = ref({ data: [], total: 0, last_page: 1, per_page: 20 });
const loading = ref(false);

async function getItems(url = "/product-types") {
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

function changePage(selectedPage) {
  router.replace({ name: route.name, query: { page: selectedPage } });
  getItems(`/product-types?page=${selectedPage}`);
}

const deleteItem = (id) => {
  Swal.fire({
    title: "حذف دسته‌بندی",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/product-types/${id}`);
        Swal.fire("موفق", "دسته‌بندی حذف شد", "success");
        getItems();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  currentPage.value = route.query.page ?? 1;
  getItems();
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
.type-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.type-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.type-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.type-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.type-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.type-name {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.type-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.type-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.type-info-row i {
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
}

/* ========================================= */
/* ===== موبایل کوچک (کمتر از 400px) ===== */
/* ========================================= */
@media (max-width: 399.98px) {
  .page-title {
    font-size: 1rem;
  }

  .type-card {
    padding: 12px;
  }

  .type-name {
    font-size: 0.9rem;
  }

  .type-info-row {
    font-size: 0.78rem;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .type-cards {
    display: none;
  }
}
</style>