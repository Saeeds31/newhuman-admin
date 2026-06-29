<template>
    <div class="container py-4" v-if="checkPermission(['report_products'])">
        <!-- فیلترها -->
        <div class="card mb-4">
            <div class="card-header d-flex justify-content-between align-items-center mb-3">
                <h3>
                    <i class="bi bi-reception-4"></i>
                    <span>
                        گزارش محصولات
                    </span>
                </h3>
                <b-spinner small v-if="loading"></b-spinner>
            </div>
            <div class="card-body">
                <form @submit.prevent="getReport()" class="row g-3">
                    <div class="col-md-2">
                        <select v-model="filters.category_id" class="form-select">
                            <option value="">همه دسته‌بندی‌ها</option>
                            <option v-for="cat in categories" :key="cat.id" :value="cat.id">
                                {{ cat.name }}
                            </option>
                        </select>
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.status" class="form-select">
                            <option value="">همه وضعیت‌ها</option>
                            <option value="published">منتشر شده</option>
                            <option value="unpublished">منتشر نشده</option>
                        </select>
                    </div>
                    <div class="col-md-2">
                        <input v-model="filters.price_min" type="number" class="form-control"
                            placeholder="حداقل قیمت" />
                    </div>
                    <div class="col-md-2">
                        <input v-model="filters.price_max" type="number" class="form-control"
                            placeholder="حداکثر قیمت" />
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.in_stock" class="form-select">
                            <option value="">همه</option>
                            <option value="1">دارای موجودی</option>
                        </select>
                    </div>
                    <div class="col-md-2">
                        <select v-model="filters.has_discount" class="form-select">
                            <option value="">همه</option>
                            <option value="1">دارای تخفیف</option>
                        </select>
                    </div>
                    <div class="col-md-2">
                        <date-picker display-format="jYYYY/jMM/jDD" placeholder="از تاریخ" format="YYYY-MM-DD"
                            v-model="filters.date_from"></date-picker>
                    </div>
                    <div class="col-md-2">
                        <date-picker display-format="jYYYY/jMM/jDD" placeholder="تا تاریخ" format="YYYY-MM-DD"
                            v-model="filters.date_to"></date-picker>
                    </div>
                    <div class="col-12">
                        <button type="submit" class="btn btn-primary w-100">
                            <i class="bi bi-save2"></i>
                            <span>
                                اعمال فیلتر
                            </span>
                        </button>
                    </div>
                </form>
            </div>
        </div>

        <!-- جدول گزارش -->
        <div class="card">
            <div class="card-body">
                <h2 class="h5 mb-3">گزارش محصولات</h2>

                <!-- لودر -->
                <div v-if="loading" class="text-center py-5">
                    <div class="spinner-border text-primary" role="status">
                        <span class="visually-hidden">در حال بارگذاری...</span>
                    </div>
                </div>

                <!-- جدول -->
                <div v-else>
                    <div class="table-responsive">
                        <table class="table table-striped align-middle text-center">
                            <thead class="table-light">
                                <tr>
                                    <th>شناسه</th>
                                    <th>عنوان</th>
                                    <th>قیمت</th>
                                    <th>موجودی</th>
                                    <th>وضعیت</th>
                                    <th>تخفیف</th>
                                    <th>تاریخ ایجاد</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="product in tableData.data" :key="product.id">
                                    <td>{{ product.id }}</td>
                                    <td>{{ product.title }}</td>
                                    <td>{{ product.price.toLocaleString() }} تومان</td>
                                    <td>
                                        <span :class="product.stock > 0 ? 'text-success' : 'text-danger'">
                                            {{ product.stock > 0 ? product.stock : "ناموجود" }}
                                        </span>
                                    </td>
                                    <td>{{ translateStatus(product.status) }}</td>
                                    <td>
                                        <span v-if="product.discount_value">
                                            {{ product.discount_value }}
                                            {{ product.discount_type === "percent" ? "%" : "تومان" }}
                                        </span>
                                        <span v-else>-</span>
                                    </td>
                                    <td>
                                        {{ new Date(product.created_at).toLocaleDateString("fa-IR") }}
                                    </td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <!-- صفحه‌بندی -->
                    <b-pagination v-model="currentPage" :total-rows="tableData.total" v-if="tableData.last_page != 1"
                        :per-page="tableData.per_page" @Update:modelValue="changePage" align="center"
                        class="mt-3"></b-pagination>

                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import axios from "axios";
import { ref, onMounted } from "vue";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const filters = ref({
    category_id: "",
    status: "",
    price_min: "",
    price_max: "",
    in_stock: "",
    date_from: "",
    date_to: "",
    has_discount: "",
});

const categories = ref([]); // لیست دسته‌ها (از API پر میشه)
const tableData = ref({ data: [] });
const loading = ref(false);

const getReport = async (page = 1) => {
    try {
        loading.value = true;
        const { data } = await axios.get("/reports/products?page=" + page, {
            params: { ...filters.value },
        });
        tableData.value = data;
    } catch (error) {
        console.error("خطا در گرفتن گزارش:", error);
    } finally {
        loading.value = false;
    }
};

const changePage = (page) => {
    if (page > 0) {
        getReport(page);
    }
};

const translateStatus = (status) => {
    const map = {
        published: "منتشر شده",
        unpublished: "منتشر نشده",
    };
    return map[status] || status;
};

onMounted(() => {
    getReport();
    // دسته‌بندی‌ها رو هم بیار (فرض می‌کنم API آماده داری)
    axios.get("/categories").then((res) => {
        categories.value = res.data;
    });
});
</script>