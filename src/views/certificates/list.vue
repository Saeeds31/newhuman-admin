<template>
    <div class="container mt-4" v-if="checkPermission(['certificate_view'])">
        <div class="card mb-2">
            <div class="card-header d-flex justify-content-between align-items-center mb-3">
                <h3>
                    <i class="bi bi-award"></i>
                    <span>مدیریت گواهینامه‌ها</span>
                </h3>
                <router-link to="/certificates/create" class="btn btn-success">
                    <i class="bi bi-plus"></i>
                    <span>افزودن گواهینامه</span>
                </router-link>
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
            <div class="card-body">
                <div v-if="loading" class="text-center py-5">
                    <div class="spinner-border text-primary" role="status">
                        <span class="visually-hidden">در حال بارگذاری...</span>
                    </div>
                </div>

                <div v-else>
                    <table class="table table-bordered table-striped">
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
                                    <!-- <router-link :to="`/certificates/${certificate.id}/edit`"
                                        class="btn btn-sm btn-warning me-2">
                                        <i class="bi bi-pen"></i>
                                        <span> ویرایش</span>
                                    </router-link> -->
                                    <button class="btn btn-sm btn-danger" @click="deleteCertificate(certificate.id)">
                                        <i class="bi bi-trash3-fill"></i>
                                        <span>حذف</span>
                                    </button>
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

                    <!-- صفحه‌بندی -->
                    <b-pagination v-model="currentPage" :total-rows="certificates.total"
                        v-if="certificates.last_page != 1" :per-page="certificates.per_page"
                        @update:modelValue="changePage" align="center" class="mt-3" />
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
.table th,
.table td {
    vertical-align: middle;
}

.badge {
    font-size: 0.85rem;
    padding: 5px 10px;
}
</style>