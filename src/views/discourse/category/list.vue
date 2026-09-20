<template>
    <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['discourse_view'])">
        <div class="card mb-2 header-card">
            <div class="card-header">
                <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
                    <h3 class="mb-0 page-title">
                        <i class="bi bi-list-stars"></i>
                        <span>مدیریت دسته بندی</span>
                    </h3>
                    <router-link to="/discourse/categories/create" class="btn btn-primary add-btn">
                        <i class="bi bi-plus"></i>
                        <span>افزودن دسته بندی</span>
                    </router-link>
                </div>
            </div>
        </div>

        <!-- ===== نمایش جدول در دسکتاپ ===== -->
        <div class="card d-none d-md-block">
            <div class="card-body p-2 p-md-3">
                <b-table class="table table-bordered table-striped mb-0" striped hover :items="categories.data" :fields="fields">
                    <template #cell(actions)="data">
                        <div class="d-flex flex-wrap gap-1">
                            <router-link :to="`/discourse/categories/${data.item.id}/edit`" class="btn btn-sm btn-warning">
                                <i class="bi bi-pen"></i>
                                <span>ویرایش</span>
                            </router-link>
                            <button class="btn btn-sm btn-danger" @click="confirmDelete(data.item.id)">
                                <i class="bi bi-trash3-fill"></i>
                                <span>حذف</span>
                            </button>
                        </div>
                    </template>
                </b-table>
            </div>
        </div>

        <!-- ===== نمایش کارتی در موبایل ===== -->
        <div class="card d-md-none">
            <div class="card-body p-2">
                <div class="discourse-cards">
                    <div
                        v-for="cat in categories.data"
                        :key="cat.id"
                        class="discourse-card"
                    >
                        <div class="discourse-card-header">
                            <div class="discourse-id-badge">#{{ cat.id }}</div>
                            <div class="discourse-title">{{ cat.title }}</div>
                        </div>

                        <div class="discourse-card-body">
                            <div class="discourse-info-row">
                                <i class="bi bi-link-45deg"></i>
                                <span class="info-label">اسلاگ:</span>
                                <span class="info-value">{{ cat.slug }}</span>
                            </div>
                        </div>

                        <div class="discourse-card-actions">
                            <router-link :to="`/discourse/categories/${cat.id}/edit`" class="btn btn-sm btn-warning flex-fill">
                                <i class="bi bi-pen"></i>
                                <span>ویرایش</span>
                            </router-link>
                            <button class="btn btn-sm btn-danger flex-fill" @click="confirmDelete(cat.id)">
                                <i class="bi bi-trash3-fill"></i>
                                <span>حذف</span>
                            </button>
                        </div>
                    </div>

                    <!-- حالت خالی -->
                    <div v-if="!categories.data || categories.data.length === 0" class="text-center py-5 text-muted">
                        <i class="bi bi-inbox fs-1 d-block mb-2"></i>
                        <p>دسته‌بندی‌ای یافت نشد</p>
                    </div>
                </div>
            </div>
        </div>

        <b-pagination v-model="currentPage" :total-rows="categories.total" v-if="categories.last_page != 1"
            :per-page="categories.per_page" @Update:modelValue="fetchCategories" align="center"
            class="mt-3 pagination-responsive"></b-pagination>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import Swal from 'sweetalert2'
import { useRoute, useRouter } from 'vue-router'
let router = useRouter();
let route = useRoute();
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const categories = ref({
    data: [],
    total: 0,
    per_page: 10,
    current_page: 1,
})
const currentPage = ref(1)

const fields = [
    { key: 'id', label: 'شناسه' },
    { key: 'title', label: 'عنوان' },
    { key: 'slug', label: 'اسلاگ' },
    { key: 'actions', label: 'عملیات' },
]

const fetchCategories = async (page = 1) => {
    try {
        router.replace({ name: route.name, query: { page: page } })
        const res = await axios.get(`/discourse-categories?page=${page}`)
        categories.value = res.data;
        currentPage.value = res.data.current_page
    } catch (error) {
        console.error(error)
    }
}

const confirmDelete = (id) => {
    Swal.fire({
        title: 'آیا مطمئن هستید?',
        text: "این عملیات بازگشت پذیر نیست!",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#d33',
        cancelButtonColor: '#3085d6',
        confirmButtonText: 'بله انجام شود!',
        cancelButtonText: 'لغو',
    }).then((result) => {
        if (result.isConfirmed) {
            deleteCategory(id)
        }
    })
}

const deleteCategory = async (id) => {
    try {
        await axios.delete(`/discourse-categories/${id}`)
        Swal.fire('پاک شد!', 'با موفقیت حذف شد.', 'success')
        fetchCategories(currentPage.value)
    } catch (error) {
        console.error(error)
        Swal.fire('Error!', error.response.data.message ?? 'خطایی در حذف رخ داد', 'error')
    }
}

onMounted(() => {
    fetchCategories()
})


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
.discourse-cards {
    display: flex;
    flex-direction: column;
    gap: 12px;
}

.discourse-card {
    background: #fff;
    border: 1px solid #e9ecef;
    border-radius: 12px;
    padding: 14px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
    transition: all 0.2s ease;
}

.discourse-card:hover {
    box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
    transform: translateY(-2px);
}

.discourse-card-header {
    display: flex;
    align-items: center;
    gap: 10px;
    padding-bottom: 10px;
    border-bottom: 1px solid #f0f0f0;
    margin-bottom: 10px;
}

.discourse-id-badge {
    background: linear-gradient(135deg, #6c5ce7, #a29bfe);
    color: white;
    font-size: 0.75rem;
    font-weight: 700;
    padding: 4px 10px;
    border-radius: 20px;
    flex-shrink: 0;
}

.discourse-title {
    font-weight: 700;
    color: #2d3436;
    font-size: 1rem;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.discourse-card-body {
    display: flex;
    flex-direction: column;
    gap: 8px;
    margin-bottom: 12px;
}

.discourse-info-row {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.85rem;
}

.discourse-info-row i {
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

.discourse-card-actions {
    display: flex;
    gap: 6px;
    padding-top: 10px;
    border-top: 1px solid #f0f0f0;
}

.discourse-card-actions .btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 4px;
    font-size: 0.75rem;
    padding: 6px 8px;
    white-space: nowrap;
}

.discourse-card-actions .btn span {
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
    .discourse-card-actions .btn span {
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

    .discourse-card {
        padding: 12px;
    }

    .discourse-title {
        font-size: 0.9rem;
    }

    .discourse-info-row {
        font-size: 0.78rem;
    }

    .discourse-card-actions .btn {
        font-size: 0.7rem;
        padding: 5px 6px;
    }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
    .discourse-cards {
        display: none;
    }
}
</style>