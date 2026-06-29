<template>
    <div class="container mt-4  orders-page" v-if="checkPermission(['order_view'])">
   
        <div class="card mb-3">
            <div class="card-header d-flex justify-content-between align-items-center mb-3">
                <h3>
                    <i class="bi bi-list-check"></i>
                    <span>
                        مدیریت سفارش‌ها
                    </span>
                </h3>
                <router-link to="/orders/create" class="btn btn-primary">
                    <i class="bi bi-plus"></i>
                    <span>
                        افزودن سفارش
                    </span>
                </router-link>
            </div>
            <div class="card-body row g-2">
                <div class="col-md-3">
                    <input v-model="filters.search" @input="getOrders" type="text" class="form-control"
                        placeholder="جستجو (کاربر یا شماره سفارش)" />
                </div>
                <div class="col-md-2">
                    <select v-model="filters.status" @change="getOrders" class="form-select">
                        <option value="">همه وضعیت‌ها</option>
                        <option value="pending">در انتظار</option>
                        <option value="reserved">رزرو شده</option>
                        <option value="processing">در حال پردازش</option>
                        <option value="shipped">ارسال شده</option>
                        <option value="completed">تکمیل شده</option>
                        <option value="canceled">لغو شده</option>
                        <option value="returned">مرجوعی</option>
                    </select>
                </div>
                <div class="col-md-2">
                    <select v-model="filters.payment_status" @change="getOrders" class="form-select">
                        <option value="">همه پرداخت‌ها</option>
                        <option value="pending">در انتظار پرداخت</option>
                        <option value="paid">پرداخت شده</option>
                        <option value="failed">ناموفق</option>
                        <option value="refunded">برگشت داده شده</option>
                    </select>
                </div>
                <div class="col-md-2">
                    <select v-model="filters.payment_method" @change="getOrders" class="form-select">
                        <option value="">روش پرداخت</option>
                        <option value="online">پرداخت آنلاین</option>
                        <option value="wallet">کیف پول</option>
                        <option value="cod">پرداخت در محل</option>
                    </select>
                </div>
            </div>
        </div>

        <!-- Table -->
        <div class="card">
            <div class="card-body table-responsive">
                <table class="table table-bordered align-middle text-center">
                    <thead>
                        <tr>
                            <th>#</th>
                            <th>کاربر</th>
                            <th>آدرس</th>
                            <th>روش ارسال</th>
                            <th>مبلغ کل</th>
                            <th>وضعیت سفارش</th>
                            <th>وضعیت پرداخت</th>
                            <th>روش پرداخت</th>
                            <th>عملیات</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-if="loading">
                            <td colspan="9" class="text-center">
                                <div class="spinner-border" role="status"></div>
                            </td>
                        </tr>
                        <tr v-else v-for="order in orders.data" :key="order.id">
                            <td>{{ order.id }}</td>
                            <td>{{ order.user?.full_name ?? "-" }}</td>
                            <td>{{ order.address?.address_line ?? "-" }}</td>
                            <td>{{ order.shipping_method?.name ?? "-" }}</td>
                            <td>{{ order.total }} تومان</td>
                            <td>
                                <span class="badge" :class="statusBadge(order.status)">
                                    {{ statusText(order.status) }}
                                </span>
                            </td>
                            <td>
                                <span class="badge" :class="paymentStatusBadge(order.payment_status)">
                                    {{ paymentStatusText(order.payment_status) }}
                                </span>
                            </td>
                            <td>{{ paymentMethodText(order.payment_method) }}</td>
                            <td>
                                <router-link :to="`/orders/${order.id}`" class="btn btn-sm btn-info">
                                    جزئیات
                                </router-link>
                            </td>
                        </tr>
                        <tr v-if="!loading && orders.data.length === 0">
                            <td colspan="9" class="text-center">هیچ سفارشی یافت نشد</td>
                        </tr>
                    </tbody>
                </table>
            </div>
        </div>

        <b-pagination v-model="currentPage" :total-rows="orders.total" v-if="orders.last_page != 1"
            :per-page="orders.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>


    </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const orders = ref({ data: [] });
const loading = ref(false);
const filters = ref({
    search: "",
    status: "",
    payment_status: "",
    payment_method: "",
});
const currentPage = ref(1);

const getOrders = async (page = 1) => {
    loading.value = true;
    try {
        const response = await axios.get("/orders", {
            params: {
                page,
                ...filters.value,
            },
        });
        orders.value = response.data.data;
        currentPage.value = page;
    } finally {
        loading.value = false;
    }
};

const changePage = (page) => {
    if (page) getOrders(page);
    else currentUrl = "/articles"
};
// helpers
const statusText = (status) => {
    const map = {
        pending: "در انتظار",
        reserved: "رزرو شده",
        processing: "در حال پردازش",
        shipped: "ارسال شده",
        completed: "تکمیل شده",
        canceled: "لغو شده",
        returned: "مرجوعی",
    };
    return map[status] ?? status;
};

const statusBadge = (status) => {
    const map = {
        pending: "bg-secondary",
        reserved: "bg-warning text-dark",
        processing: "bg-info",
        shipped: "bg-primary",
        completed: "bg-success",
        canceled: "bg-danger",
        returned: "bg-dark",
    };
    return map[status] ?? "bg-secondary";
};

const paymentStatusText = (status) => {
    const map = {
        pending: "در انتظار پرداخت",
        paid: "پرداخت شده",
        failed: "ناموفق",
        refunded: "برگشت داده شده",
    };
    return map[status] ?? status;
};

const paymentStatusBadge = (status) => {
    const map = {
        pending: "bg-warning text-dark",
        paid: "bg-success",
        failed: "bg-danger",
        refunded: "bg-secondary",
    };
    return map[status] ?? "bg-secondary";
};

const paymentMethodText = (method) => {
    const map = {
        online: "پرداخت آنلاین",
        wallet: "کیف پول",
        cod: "پرداخت در محل",
    };
    return map[method] ?? method;
};

onMounted(() => {
    getOrders();
});
</script>