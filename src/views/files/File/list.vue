<template>
  <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['file_view'])">
    <div class="card mb-2 header-card">
      <div class="card-header">
        <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
          <h3 class="mb-0 page-title">
            <i class="bi bi-files"></i>
            <span>مدیریت فایل</span>
          </h3>
          <router-link v-if="checkPermission(['file_store'])" to="/files/create" class="btn btn-success add-btn">
            <i class="bi bi-plus"></i>
            <span>افزودن فایل</span>
          </router-link>
        </div>
      </div>
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
                  <th>عنوان</th>
                  <th>عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="file in files.data" :key="file.id">
                  <td>{{ file.id }}</td>
                  <td>{{ file.title }}</td>
                  <td>
                    <div class="d-flex flex-wrap gap-1">
                      <router-link v-if="checkPermission(['file_update'])" :to="`/files/${file.id}/edit`"
                        class="btn btn-sm btn-warning">
                        <i class="bi bi-pen"></i>
                        <span>ویرایش</span>
                      </router-link>
                      <button class="btn btn-sm btn-danger" v-if="checkPermission(['file_delete'])"
                        @click="deletefile(file.id)">
                        <i class="bi bi-trash3-fill"></i>
                        <span>حذف</span>
                      </button>
                      <button class="btn btn-sm btn-primary" @click="copyFile(file.file)">
                        <i class="bi bi-copy"></i>
                        <span>کپی</span>
                      </button>
                    </div>
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- ===== نمایش کارتی در موبایل ===== -->
          <div class="d-md-none file-cards">
            <div
              v-for="file in files.data"
              :key="file.id"
              class="file-card"
            >
              <div class="file-card-header">
                <div class="file-id-badge">#{{ file.id }}</div>
                <div class="file-title">{{ file.title }}</div>
              </div>

              <div class="file-card-actions">
                <router-link v-if="checkPermission(['file_update'])" :to="`/files/${file.id}/edit`"
                  class="btn btn-sm btn-warning flex-fill">
                  <i class="bi bi-pen"></i>
                  <span>ویرایش</span>
                </router-link>
                <button class="btn btn-sm btn-danger flex-fill" v-if="checkPermission(['file_delete'])"
                  @click="deletefile(file.id)">
                  <i class="bi bi-trash3-fill"></i>
                  <span>حذف</span>
                </button>
                <button class="btn btn-sm btn-primary flex-fill" @click="copyFile(file.file)">
                  <i class="bi bi-copy"></i>
                  <span>کپی</span>
                </button>
              </div>
            </div>

            <!-- حالت خالی -->
            <div v-if="!files.data || files.data.length === 0" class="text-center py-5 text-muted">
              <i class="bi bi-inbox fs-1 d-block mb-2"></i>
              <p>فایلی یافت نشد</p>
            </div>
          </div>

          <!-- Pagination -->
          <b-pagination v-model="currentPage" :total-rows="files.total" v-if="files.last_page != 1"
            :per-page="files.per_page" @Update:modelValue="changePage" align="center"
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
import { useRoute, useRouter, } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
const store = useAdmin();
const checkPermission = store.checkPermission;
const currentPage = ref(1)
const router = useRouter();
const route = useRoute();
const files = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ search: "", status: "" });
let currentUrl = "/files";

async function getFiles(url = currentUrl) {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    files.value = data.data;
    currentPage.value = data.current_page
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};
async function copyFile(path) {
  let text = window.baseImageAddress + path;
  try {
    await navigator.clipboard.writeText(text)
    toast.success('آدرس فایل کپی شد ✅')
  } catch (err) {
    let text = window.baseImageAddress + path;
    if (text) {
      const textarea = document.createElement('textarea')
      textarea.value = sheba
      document.body.appendChild(textarea)
      textarea.select()
      document.execCommand('copy')
      document.body.removeChild(textarea)
      toast.success('آدرس فایل کپی شد ✅')
    } else {
      console.log('خطا در کپی کردن')
    }
  }
}


function changePage(selectedPage) {
  if (selectedPage) {
    router.replace({ name: route.name, query: { page: selectedPage } })
    getFiles(`${currentUrl}?page=${selectedPage}`)
  }
  else currentUrl = "/portfolios"
}
const deletefile = (id) => {
  Swal.fire({
    title: "حذف نمونه کار",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/files/${id}`);
        Swal.fire("موفق", "نمونه کار حذف شد", "success");
        getFiles();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  currentPage.value = route.query.page ?? 1;
  getFiles(`${currentUrl}?page=${currentPage.value}`);
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
.file-cards {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

.file-card {
  background: #fff;
  border: 1px solid #e9ecef;
  border-radius: 12px;
  padding: 14px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
  transition: all 0.2s ease;
}

.file-card:hover {
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
  transform: translateY(-2px);
}

.file-card-header {
  display: flex;
  align-items: center;
  gap: 10px;
  padding-bottom: 10px;
  border-bottom: 1px solid #f0f0f0;
  margin-bottom: 10px;
}

.file-id-badge {
  background: linear-gradient(135deg, #6c5ce7, #a29bfe);
  color: white;
  font-size: 0.75rem;
  font-weight: 700;
  padding: 4px 10px;
  border-radius: 20px;
  flex-shrink: 0;
}

.file-title {
  font-weight: 700;
  color: #2d3436;
  font-size: 1rem;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.file-card-actions {
  display: flex;
  gap: 6px;
  padding-top: 10px;
  border-top: 1px solid #f0f0f0;
  flex-wrap: wrap;
}

.file-card-actions .btn {
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 4px;
  font-size: 0.75rem;
  padding: 6px 8px;
  white-space: nowrap;
}

.file-card-actions .btn span {
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

  /* نمایش label دکمه‌ها در موبایل */
  .file-card-actions .btn span {
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

  .file-card {
    padding: 12px;
  }

  .file-title {
    font-size: 0.9rem;
  }

  .file-card-actions .btn {
    font-size: 0.7rem;
    padding: 5px 6px;
  }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
  .file-cards {
    display: none;
  }
}
</style>