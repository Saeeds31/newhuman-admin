<template>
    <div class="container mt-3 mt-md-4 px-2 px-md-3" v-if="checkPermission(['certificate_view'])">
        <div class="card mb-2 header-card">
            <div class="card-header">
                <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
                    <h3 class="mb-0 page-title">
                        <i class="bi bi-award"></i>
                        <span>مدیریت گواهینامه‌ها</span>
                    </h3>
                    <router-link to="/certificates/create" class="btn btn-success add-btn">
                        <i class="bi bi-plus"></i>
                        <span>افزودن گواهینامه</span>
                    </router-link>
                </div>
            </div>
            <!-- <div class="card-body">
                <form @submit.prevent="getCertificates()">
                    <div class="row g-2">
                        <div class="col-md-3">
                            <input v-model="filters.user_id" type="text" class="form-control"
                                placeholder="شناسه کاربر" />
                        </div>
                        <div class="col-md-3">
                            <input v-model="filters.product_id" type="text" class="form-control"
                                placeholder="شناسه محصول" />
                        </div>
                        <div class="col-md-4">
                            <input v-model="filters.number" type="text" class="form-control"
                                placeholder="شماره گواهینامه" />
                        </div>
                        <div class="col-md-2">
                            <button class="btn btn-primary w-100" type="submit">جستجو</button>
                        </div>
                    </div>
                </form>
            </div> -->
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
                                    <th>شماره</th>
                                    <th>کاربر</th>
                                    <th>محصول</th>
                                    <th>تصویر</th>
                                    <th>تاریخ دریافت</th>
                                    <th>عملیات</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="certificate in certificates.data" :key="certificate.id">
                                    <td>{{ certificate.id }}</td>
                                    <td>
                                        <span class="badge bg-info">{{ certificate.number }}</span>
                                    </td>
                                    <td>{{ certificate.user?.full_name || 'نامشخص' }}</td>
                                    <td>{{ certificate.product?.title || 'نامشخص' }}</td>
                                    <td>
                                        <img v-if="certificate.image" :src="pathresolver(certificate.image)" alt="گواهینامه"
                                            width="50" height="50" class="rounded" style="object-fit: cover;" />
                                        <span v-else class="text-muted">بدون تصویر</span>
                                    </td>
                                    <td>{{ certificate.date_acquisition || 'نامشخص' }}</td>
                                    <td>
                                        <div class="d-flex flex-wrap gap-1">
                                            <!-- <router-link :to="`/certificates/${certificate.id}/edit`"
                                                class="btn btn-sm btn-warning">
                                                <i class="bi bi-pen"></i>
                                                <span>ویرایش</span>
                                            </router-link> -->
                                            <button class="btn btn-sm btn-danger" @click="deleteCertificate(certificate.id)">
                                                <i class="bi bi-trash3-fill"></i>
                                                <span>حذف</span>
                                            </button>
                                        </div>
                                    </td>
                                </tr>
                                <tr v-if="!certificates.data || certificates.data.length === 0">
                                    <td colspan="7" class="text-center text-muted py-4">
                                        <i class="bi bi-inbox"></i>
                                        <span class="mx-2">هیچ گواهینامه‌ای یافت نشد</span>
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <!-- ===== نمایش کارتی در موبایل ===== -->
                    <div class="d-md-none certificate-cards">
                        <div
                            v-for="certificate in certificates.data"
                            :key="certificate.id"
                            class="certificate-card"
                        >
                            <div class="certificate-card-header">
                                <div class="certificate-id-badge">#{{ certificate.id }}</div>
                                <div class="certificate-number">
                                    <span class="badge bg-info">{{ certificate.number }}</span>
                                </div>
                            </div>

                            <div class="certificate-card-image" v-if="certificate.image">
                                <img :src="pathresolver(certificate.image)" alt="گواهینامه" class="certificate-image">
                            </div>

                            <div class="certificate-card-body">
                                <div class="certificate-info-row">
                                    <i class="bi bi-person"></i>
                                    <span class="info-label">کاربر:</span>
                                    <span class="info-value">{{ certificate.user?.full_name || 'نامشخص' }}</span>
                                </div>
                                <div class="certificate-info-row">
                                    <i class="bi bi-box"></i>
                                    <span class="info-label">محصول:</span>
                                    <span class="info-value">{{ certificate.product?.title || 'نامشخص' }}</span>
                                </div>
                                <div class="certificate-info-row">
                                    <i class="bi bi-calendar"></i>
                                    <span class="info-label">تاریخ دریافت:</span>
                                    <span class="info-value">{{ certificate.date_acquisition || 'نامشخص' }}</span>
                                </div>
                            </div>

                            <div class="certificate-card-actions">
                                <!-- <router-link :to="`/certificates/${certificate.id}/edit`" class="btn btn-sm btn-warning flex-fill">
                                    <i class="bi bi-pen"></i>
                                    <span>ویرایش</span>
                                </router-link> -->
                                <button class="btn btn-sm btn-danger flex-fill" @click="deleteCertificate(certificate.id)">
                                    <i class="bi bi-trash3-fill"></i>
                                    <span>حذف</span>
                                </button>
                            </div>
                        </div>

                        <!-- حالت خالی -->
                        <div v-if="!certificates.data || certificates.data.length === 0" class="text-center py-5 text-muted">
                            <i class="bi bi-inbox fs-1 d-block mb-2"></i>
                            <p>هیچ گواهینامه‌ای یافت نشد</p>
                        </div>
                    </div>

                    <!-- صفحه‌بندی -->
                    <b-pagination v-model="currentPage" :total-rows="certificates.total"
                        v-if="certificates.last_page != 1" :per-page="certificates.per_page"
                        @update:modelValue="changePage" align="center" class="mt-3 pagination-responsive" />
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
import { BPagination } from 'bootstrap-vue-3';

const store = useAdmin();
const checkPermission = store.checkPermission;

const currentPage = ref(1);
const certificates = ref({ data: [], total: 0, last_page: 1, per_page: 15, current_page: 1 });
const loading = ref(false);
// const filters = ref({ user_id: "", product_id: "", number: "" });
let currentUrl = "/certificates";
function pathresolver(image) {
    return window.baseImageAddress + image
}
async function getCertificates(url) {
    loading.value = true;
    try {
        // const { data } = await axios.get(url, { params: filters.value });
        const { data } = await axios.get(url);
        certificates.value = data.data;
        currentPage.value = data.data.current_page;
    } catch (err) {
        console.error(err);
        toast.error('خطا در دریافت اطلاعات ❌');
    } finally {
        loading.value = false;
    }
}

const changePage = (page) => {
    if (page) getCertificates(`${currentUrl}?page=${page}`);
    else currentUrl = "/certificates";
};

const deleteCertificate = (id) => {
    Swal.fire({
        title: "حذف گواهینامه",
        text: "آیا مطمئن هستید؟",
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "بله، حذف شود",
        cancelButtonText: "انصراف",
    }).then(async (result) => {
        if (result.isConfirmed) {
            try {
                await axios.delete(`/certificates/${id}`);
                Swal.fire("موفق", "گواهینامه حذف شد", "success");
                getCertificates(currentUrl);
            } catch (err) {
                Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
            }
        }
    });
};

onMounted(() => {
    getCertificates(currentUrl);
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

.badge {
    font-size: 0.85rem;
    padding: 5px 10px;
}

/* ===== کارت‌های موبایل ===== */
.certificate-cards {
    display: flex;
    flex-direction: column;
    gap: 12px;
}

.certificate-card {
    background: #fff;
    border: 1px solid #e9ecef;
    border-radius: 12px;
    padding: 14px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
    transition: all 0.2s ease;
}

.certificate-card:hover {
    box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
    transform: translateY(-2px);
}

.certificate-card-header {
    display: flex;
    align-items: center;
    gap: 10px;
    padding-bottom: 10px;
    border-bottom: 1px solid #f0f0f0;
    margin-bottom: 10px;
}

.certificate-id-badge {
    background: linear-gradient(135deg, #6c5ce7, #a29bfe);
    color: white;
    font-size: 0.75rem;
    font-weight: 700;
    padding: 4px 10px;
    border-radius: 20px;
    flex-shrink: 0;
}

.certificate-number {
    flex: 1;
}

.certificate-card-image {
    display: flex;
    justify-content: center;
    margin-bottom: 12px;
}

.certificate-image {
    max-width: 100%;
    max-height: 150px;
    border-radius: 8px;
    border: 1px solid #e9ecef;
    object-fit: cover;
}

.certificate-card-body {
    display: flex;
    flex-direction: column;
    gap: 8px;
    margin-bottom: 12px;
}

.certificate-info-row {
    display: flex;
    align-items: center;
    gap: 8px;
    font-size: 0.85rem;
}

.certificate-info-row i {
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

.certificate-card-actions {
    display: flex;
    gap: 6px;
    padding-top: 10px;
    border-top: 1px solid #f0f0f0;
}

.certificate-card-actions .btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 4px;
    font-size: 0.75rem;
    padding: 6px 8px;
    white-space: nowrap;
}

.certificate-card-actions .btn span {
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
    .certificate-card-actions .btn span {
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

    .certificate-card {
        padding: 12px;
    }

    .certificate-info-row {
        font-size: 0.78rem;
    }

    .certificate-card-actions .btn {
        font-size: 0.7rem;
        padding: 5px 6px;
    }

    .certificate-image {
        max-height: 120px;
    }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
    .certificate-cards {
        display: none;
    }
}
</style>