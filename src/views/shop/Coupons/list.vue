<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['coupon_view'])">

    <!-- باکس فیلتر -->
    <div class="card mb-3 header-card">
      <div class="card-body">
        <form @submit.prevent="getCoupons">
          <div class="row g-2">
            <div class="col-12 col-sm-8 col-md-4">
              <input v-model="filters.code" type="text" class="form-control search-input" placeholder="جستجو بر اساس کد کوپن" />
            </div>
            <div class="col-12 col-sm-4 col-md-2">
              <button class="btn btn-primary w-100" type="submit">جستجو</button>
            </div>
          </div>
        </form>
      </div>
    </div>

    <!-- دکمه افزودن -->
    <div class="mb-3 d-flex justify-content-stretch justify-content-sm-end">
      <router-link to="/shop/coupons/create" class="btn btn-success add-btn w-100 w-sm-auto">
        <i class="bi bi-plus"></i>
        <span>افزودن کوپن</span>
      </router-link>
    </div>

    <!-- جدول -->
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
                  <th>کد</th>
                  <th>نوع</th>
                  <th>مقدار</th>
                  <th>تاریخ شروع</th>
                  <th>تاریخ پایان</th>
                  <th>وضعیت</th>
                  <th>عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="coupon in coupons.data" :key="coupon.id">
                  <td>{{ coupon.id }}</td>
                  <td>{{ coupon.code }}</td>
                  <td>{{ coupon.type }}</td>
                  <td>{{ coupon.value }}</td>
                  <td>{{ formatDate(coupon.start_date) }}</td>
                  <td>{{ formatDate(coupon.end_date) }}</td>
                  <td>
                    <span :class="coupon.status ? 'badge bg-success' : 'badge bg-secondary'">
                      {{ coupon.status ? 'فعال' : 'غیرفعال' }}
                    </span>
                  </td>
                  <td>
                    <div class="d-flex flex-wrap gap-1">
                      <router-link :to="`/shop/coupons/${coupon.id}/edit`" class="btn btn-sm btn-warning">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button class="btn btn-sm btn-danger" @click="deleteCoupon(coupon.id)">
                        <i class="bi bi-trash3-fill"></i>
                        <span>حذف</span>
                      </button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- ===== نمایش کارتی در موبایل ===== -->
          <div class="d-md-none coupon-cards">
            <div
              v-for="coupon in coupons.data"
              :key="coupon.id"
              class="coupon-card"
            >
              <div class="coupon-card-header">
                <div class="coupon-id-badge">#{{ coupon.id }}</div>
                <div class="coupon-code">{{ coupon.code }}</div>
                <span :class="coupon.status ? 'badge bg-success' : 'badge bg-secondary'">
                  {{ coupon.status ? 'فعال' : 'غیرفعال' }}
                </span>
              </div>

              <div class="coupon-card-body">
                <div class="coupon-info-row">
                  <i class="bi bi-tag"></i>
                  <span class="info-label">نوع:</span>
                  <span class="info-value">{{ coupon.type }}</span>
                </div>
                <div class="coupon-info-row">
                  <i class="bi bi-currency-dollar"></i>
                  <span class="info-label">مقدار:</span>
                  <span class="info-value">{{ coupon.value }}</span>
                </div>
                <div class="coupon-info-row">
                  <i class="bi bi-calendar-plus"></i>
                  <span class="info-label">شروع:</span>
                  <span class="info-value">{{ formatDate(coupon.start_date) }}</span>
                </div>
                <div class="coupon-info-row">
                  <i class="bi bi-calendar-x"></i>
                  <span class="info-label">پایان:</span>
                  <span class="info-value">{{ formatDate(coupon.end_date) }}</span>
                </div>
              </div>

              <div class="coupon-card-actions">
                <router-link :to="`/shop/coupons/${coupon.id}/edit`" class="btn btn-sm btn-warning flex-fill">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button class="btn btn-sm btn-danger flex-fill" @click="deleteCoupon(coupon.id)">
                  <i class="bi bi-trash3-fill"></i>
                  <span>حذف</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!coupons.data || coupons.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>کوپنی یافت نشد</p>
            </div>
          </div>

          <!-- صفحه بندی -->
          <b-pagination v-model="currentPage" :total-rows="coupons.total" v-if="coupons.last_page != 1"
            :per-page="coupons.per_page" @Update:modelValue="changePage" align="center"
            class="mt-3 pagination-responsive"></b-pagination>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const coupons = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ code: "" });
let currentUrl = "/coupons";

const getCoupons = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    coupons.value = data.data;
    currentUrl = url;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const changePage = (page) => {
  if (page) getCoupons(`${currentUrl}?page=${page}`);
  else currentUrl = "/copons"
};

const deleteCoupon = (id) => {
  Swal.fire({
    title: "حذف کوپن",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/coupons/${id}`);
        Swal.fire("موفق", "کوپن حذف شد", "success");
        getCoupons();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

const formatDate = (date) => {
  return new Date(date).toLocaleDateString("fa-IR");
};

onMounted(() => {
  getCoupons();
});
</script>

<style scoped>
/* ===== هدر صفحه ===== */
.header-card .card-header {
  padding: 16px 20px;
  background: transparent;
  border-bottom: 2px solid #f8f9fa;
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
.coupon-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.coupon-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.coupon-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.coupon-card-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.coupon-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.coupon-code {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  flex: 1;
  min-width: 0;
}

.coupon-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.coupon-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.coupon-info-row i {
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

.coupon-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.coupon-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.coupon-card-actions .btn span {
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

  .search-input {
    padding: 9px 12px;
    font-size: 0.9rem;
  }

  /* نمایش label دکمه‌ها در موبایل */
  .coupon-card-actions .btn span {
    display: inline;
  }
}

/* ========================================= */
/* ===== موبایل کوچک (کمتر از 400px) ===== */
/* ========================================= */
@media (max-width: 399.98px) {
  .coupon-card {
    padding: 12px;
  }

  .coupon-code {
    font-size: 0.9rem;
  }

  .coupon-info-row {
    font-size: 0.78rem;
  }

  .coupon-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .coupon-cards {
    display: none;
  }
}
</style>