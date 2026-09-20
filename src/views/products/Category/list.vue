<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3">
    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-list-nested"></i>
            <span>دسته بندی محصولات</span>
          </h3>
          <router-link to="/products/categories/create" class="btn btn-success add-btn">
            <i class="bi bi-plus"></i>
            <span>افزودن دسته‌بندی</span>
          </router-link>
        </div>
      </div>
    </div>

    <!-- جدول -->
    <div class="card">
      <div class="card-body p-2 p-md-3">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">در حال بارگذاری...</span>
          </div>
        </div>

        <div v-else>
          <!-- ===== نمایش جدول در دسکتاپ ===== -->
          <div class="table-responsive d-none d-md-block">
            <table class="table table-bordered table-striped mb-0">
              <thead>
                <tr>
                  <th>شناسه</th>
                  <th>عنوان</th>
                  <th>دسته والد</th>
                  <th>عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="cat in categories" :key="cat.id">
                  <td>{{ cat.id }}</td>
                  <td>
                    <span :style="{ 'padding-right': `${cat.level * 20}px` }">
                      {{ cat.title }}
                    </span>
                  </td>
                  <td>{{ cat.parent?.title || '---' }}</td>
                  <td>
                    <div class="d-flex flex-wrap gap-1">
                      <router-link :to="`/products/categories/${cat.id}/edit`" class="btn btn-sm btn-warning">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button class="btn btn-sm btn-danger" @click="deleteCategory(cat.id)">
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
          <div class="d-md-none category-cards">
            <div
              v-for="cat in categories"
              :key="cat.id"
              class="category-card"
            >
              <div class="category-card-header">
                <div class="category-id-badge">#{{ cat.id }}</div>
                <div class="category-title">
                  <span :style="{ 'padding-right': `${cat.level * 12}px` }">
                    {{ cat.title }}
                  </span>
                </div>
              </div>

              <div class="category-card-body">
                <div class="category-info-row">
                  <i class="bi bi-diagram-3"></i>
                  <span class="info-label">دسته والد:</span>
                  <span class="info-value">{{ cat.parent?.title || '---' }}</span>
                </div>
              </div>

              <div class="category-card-actions">
                <router-link :to="`/products/categories/${cat.id}/edit`" class="btn btn-sm btn-warning flex-fill">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button class="btn btn-sm btn-danger flex-fill" @click="deleteCategory(cat.id)">
                  <i class="bi bi-trash3-fill"></i>
                  <span>حذف</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!categories || categories.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>دسته‌بندی‌ای یافت نشد</p>
            </div>
          </div>
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
const categories = ref();
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/categories";

const getCategories = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    categories.value = flattenCategories(data.data);

  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// تابع برای مسطح کردن منوهای سلسله‌مراتبی
const flattenCategories = (categoryItems, level = 0, parent = null) => {
  let result = [];
  categoryItems.forEach((category) => {
    // اضافه کردن منوی فعلی به لیست
    result.push({
      ...category,
      level, // برای نمایش فاصله‌گذاری (indentation)
      parent, // برای دسترسی به والد
    });
    // اگر فرزندان وجود دارند، آن‌ها را هم به‌صورت بازگشتی اضافه کن
    if (category.all_children && category.all_children.length > 0) {
      result = result.concat(
        flattenCategories(category.all_children, level + 1, category)
      );
    }
  });
  return result;
};


const deleteCategory = (id) => {
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
        await axios.delete(`/categories/${id}`);
        Swal.fire("موفق", "دسته‌بندی حذف شد", "success");
        getCategories();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getCategories();
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

/* برای بهبود نمایش فاصله‌گذاری در دسته‌های فرزند */
td span {
  display: inline-block;
}

/* ===== کارت‌های موبایل ===== */
.category-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.category-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.category-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.category-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.category-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.category-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.category-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.category-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.category-info-row i {
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

.category-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.category-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.category-card-actions .btn span {
  display: none;
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
  .category-card-actions .btn span {
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

  .category-card {
    padding: 12px;
  }

  .category-title {
    font-size: 0.9rem;
  }

  .category-info-row {
    font-size: 0.78rem;
  }

  .category-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .category-cards {
    display: none;
  }
}
</style>