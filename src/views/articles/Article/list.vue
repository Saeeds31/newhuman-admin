<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['article_view'])">

    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-book-half"></i>
            <span>مدیریت مقاله</span>
          </h3>
          <router-link to="/articles/create" class="btn btn-success add-btn">
            <i class="bi bi-plus"></i>
            <span>افزودن مقاله</span>
          </router-link>
        </div>
      </div>
      <div class="card-body">
        <form @submit.prevent="getArticles()">
          <div class="row g-2">
            <div class="col-12 col-sm-8 col-md-4">
              <input v-model="filters.title" type="text" class="form-control search-input" placeholder="جستجو بر اساس عنوان" />
            </div>
            <div class="col-12 col-sm-4 col-md-2">
              <button class="btn btn-primary w-100" type="submit">جستجو</button>
            </div>
          </div>
        </form>
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
                  <th>اسلاگ</th>
                  <th>مدت زمان مطالعه</th>
                  <th>عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="article in articles.data" :key="article.id">
                  <td>{{ article.id }}</td>
                  <td>{{ article.title }}</td>
                  <td>{{ article.slug }}</td>
                  <td>{{ article.read_time }} </td>
                  <td>
                    <div class="d-flex flex-wrap gap-1">
                      <router-link :to="`/articles/${article.id}/edit`" class="btn btn-sm btn-warning">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button class="btn btn-sm btn-danger" @click="deleteArticle(article.id)">
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
          <div class="d-md-none article-cards">
            <div
              v-for="article in articles.data"
              :key="article.id"
              class="article-card"
            >
              <div class="article-card-header">
                <div class="article-id-badge">#{{ article.id }}</div>
                <div class="article-title">{{ article.title }}</div>
              </div>

              <div class="article-card-body">
                <div class="article-info-row">
                  <i class="bi bi-link-45deg"></i>
                  <span class="info-label">اسلاگ:</span>
                  <span class="info-value">{{ article.slug }}</span>
                </div>
                <div class="article-info-row">
                  <i class="bi bi-clock"></i>
                  <span class="info-label">مدت زمان مطالعه:</span>
                  <span class="info-value">{{ article.read_time }}</span>
                </div>
              </div>

              <div class="article-card-actions">
                <router-link :to="`/articles/${article.id}/edit`" class="btn btn-sm btn-warning flex-fill">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button class="btn btn-sm btn-danger flex-fill" @click="deleteArticle(article.id)">
                  <i class="bi bi-trash3-fill"></i>
                  <span>حذف</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!articles.data || articles.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>مقاله‌ای یافت نشد</p>
            </div>
          </div>

          <!-- صفحه بندی  -->
          <b-pagination v-model="currentPage" :total-rows="articles.total" v-if="articles.last_page != 1"
            :per-page="articles.per_page" @Update:modelValue="changePage" align="center"
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
const currentPage = ref(1)
const articles = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/articles";
async function getArticles(url) {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    articles.value = data.data;
    currentPage.value = data.data.current_page
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const changePage = (page) => {
  if (page) getArticles(`${currentUrl}?page=${page}`);
  else currentUrl = "/articles"
};

const deleteArticle = (id) => {
  Swal.fire({
    title: "حذف مقاله",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/articles/${id}`);
        Swal.fire("موفق", "مقاله حذف شد", "success");
        getArticles();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getArticles(currentUrl);
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

.table th,
.table td {
  vertical-align: middle;
}

.table thead th {
  background: #f8f9fa;
  font-weight: 600;
  color: #2d3436;
  white-space: nowrap;
  font-size: 0.9rem;
}

.table tbody td {
  font-size: 0.9rem;
}

/* ===== کارت‌های موبایل ===== */
.article-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.article-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.article-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.article-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.article-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.article-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.article-card-body {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 12px;
}

.article-info-row {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
}

.article-info-row i {
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

.article-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
}

.article-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.article-card-actions .btn span {
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
  .article-card-actions .btn span {
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

  .article-card {
    padding: 12px;
  }

  .article-title {
    font-size: 0.9rem;
  }

  .article-info-row {
    font-size: 0.78rem;
  }

  .article-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .article-cards {
    display: none;
  }
}
</style>