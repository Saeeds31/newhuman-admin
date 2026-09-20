<template>
    <div class="container mt-3 mt-md-4 px-2 px-md-3 orders-page" v-if="checkPermission(['order_view'])">
   
        <div class="card mb-3 header-card">
            <div class="card-header">
                <div class="d-flex flex-column flex-sm-row justify-content-between align-items-stretch align-items-sm-center gap-2 mb-3">
                    <h3 class="mb-0 page-title">
                        <i class="bi bi-list-check"></i>
                        <span>مدیریت سفارش‌ها</span>
                    </h3>
                    <router-link to="/orders/create" class="btn btn-primary add-btn">
                        <i class="bi bi-plus"></i>
                        <span>افزودن سفارش</span>
                    </router-link>
                </div>
            </div>
            <div class="card-body row g-2">
                <div class="col-12 col-sm-6 col-md-3">
                    <input v-model="filters.search" @input="getOrders" type="text" class="form-control search-input"
                        placeholder="جستجو (کاربر یا شماره سفارش)" />
                </div>
                <div class="col-12 col-sm-6 col-md-2">
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
                <div class="col-12 col-sm-6 col-md-2">
                    <select v-model="filters.payment_status" @change="getOrders" class="form-select">
                        <option value="">همه پرداخت‌ها</option>
                        <option value="pending">در انتظار پرداخت</option>
                        <option value="paid">پرداخت شده</option>
                        <option value="failed">ناموفق</option>
                        <option value="refunded">برگشت داده شده</option>
                    </select>
                </div>
                <div class="col-12 col-sm-6 col-md-2">
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
            <div class="card-body p-2 p-md-3">
                <div v-if="loading" class="text-center py-5">
                    <div class="spinner-border" role="status"></div>
                </div>

                <div v-else>
                    <!-- ===== نمایش جدول در دسکتاپ ===== -->
                    <div class="table-responsive d-none d-md-block">
                        <table class="table table-bordered align-middle text-center mb-0">
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
                                <tr v-for="order in orders.data" :key="order.id">
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
                                            <i class="bi bi-eye"></i>
                                            <span>جزئیات</span>
                                        </router-link>
                                    </td>
                                </tr>
                                <tr v-if="orders.data.length === 0">
                                    <td colspan="9" class="text-center">هیچ سفارشی یافت نشد</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                    <!-- ===== نمایش کارتی در موبایل ===== -->
                    <div class="d-md-none order-cards">
                        <div
                            v-for="order in orders.data"
                            :key="order.id"
                            class="order-card"
                        >
                            <div class="order-card-header">
                                <div class="order-id-badge">#{{ order.id }}</div>
                                <div class="order-user">{{ order.user?.full_name ?? "-" }}</div>
                            </div>

                            <div class="order-card-body">
                                <div class="order-info-row">
                                    <i class="bi bi-geo-alt"></i>
                                    <span class="info-label">آدرس:</span>
                                    <span class="info-value">{{ order.address?.address_line ?? "-" }}</span>
                                </div>
                                <div class="order-info-row">
                                    <i class="bi bi-truck"></i>
                                    <span class="info-label">روش ارسال:</span>
                                    <span class="info-value">{{ order.shipping_method?.name ?? "-" }}</span>
                                </div>
                                <div class="order-info-row">
                                    <i class="bi bi-cash-stack"></i>
                                    <span class="info-label">مبلغ کل:</span>
                                    <span class="info-value">{{ order.total }} تومان</span>
                                </div>
                                <div class="order-info-row">
                                    <i class="bi bi-toggle-on"></i>
                                    <span class="info-label">وضعیت سفارش:</span>
                                    <span class="badge" :class="statusBadge(order.status)">
                                        {{ statusText(order.status) }}
                                    </span>
                                </div>
                                <div class="order-info-row">
                                    <i class="bi bi-credit-card"></i>
                                    <span class="info-label">وضعیت پرداخت:</span>
                                    <span class="badge" :class="paymentStatusBadge(order.payment_status)">
                                        {{ paymentStatusText(order.payment_status) }}
                                    </span>
                                </div>
                                <div class="order-info-row">
                                    <i class="bi bi-wallet2"></i>
                                    <span class="info-label">روش پرداخت:</span>
                                    <span class="info-value">{{ paymentMethodText(order.payment_method) }}</span>
                                </div>
                            </div>

                            <div class="order-card-actions">
                                <router-link :to="`/orders/${order.id}`" class="btn btn-sm btn-info flex-fill">
                                    <i class="bi bi-eye"></i>
                                    <span>جزئیات</span>
                                </router-link>
                            </div>
                        </div>

                        <!-- حالت خالی -->
                        <div v-if="!orders.data || orders.data.length === 0" class="text-center py-5 text-muted">
                            <i class="bi bi-inbox fs-1 d-block mb-2"></i>
                            <p>هیچ سفارشی یافت نشد</p>
                        </div>
                    </div>

                    <!-- Pagination -->
                    <b-pagination v-model="currentPage" :total-rows="orders.total" v-if="orders.last_page != 1"
                        :per-page="orders.per_page" @Update:modelValue="changePage" align="center"
                        class="mt-3 pagination-responsive"></b-pagination>
                </div>
            </div>
        </div>
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

.badge {
    font-size: 0.8rem;
    padding: 0.35rem 0.65rem;
}

/* ===== کارت‌های موبایل ===== */
.order-cards {
    display: flex;
    flex-direction: column;
    gap: 12px;
}

.order-card {
    background: #fff;
    border: 1px solid #e9ecef;
    border-radius: 12px;
    padding: 14px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.04);
    transition: all 0.2s ease;
}

.order-card:hover {
    box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
    transform: translateY(-2px);
}

.order-card-header {
    display: flex;
    align-items: center;
    gap: 10px;
    padding-bottom: 10px;
    border-bottom: 1px solid #f0f0f0;
    margin-bottom: 10px;
}

.order-id-badge {
    background: linear-gradient(135deg, #6c5ce7, #a29bfe);
    color: white;
    font-size: 0.75rem;
    font-weight: 700;
    padding: 4px 10px;
    border-radius: 20px;
    flex-shrink: 0;
}

.order-user {
    font-weight: 700;
    color: #2d3436;
    font-size: 1rem;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
}

.order-card-body {
    display: flex;
    flex-direction: column;
    gap: 8px;
    margin-bottom: 12px;
}

.order-info-row {
    display: flex;
    align-items: flex-start;
    gap: 8px;
    font-size: 0.85rem;
    flex-wrap: wrap;
}

.order-info-row i {
    color: #6c5ce7;
    font-size: 0.95rem;
    width: 18px;
    text-align: center;
    flex-shrink: 0;
    margin-top: 2px;
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

.order-card-actions {
    display: flex;
    gap: 6px;
    padding-top: 10px;
    border-top: 1px solid #f0f0f0;
}

.order-card-actions .btn {
    display: inline-flex;
    align-items: center;
    justify-content: center;
    gap: 4px;
    font-size: 0.75rem;
    padding: 6px 8px;
    white-space: nowrap;
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
}

/* ========================================= */
/* ===== موبایل کوچک (کمتر از 400px) ===== */
/* ========================================= */
@media (max-width: 399.98px) {
    .page-title {
        font-size: 1rem;
    }

    .order-card {
        padding: 12px;
    }

    .order-user {
        font-size: 0.9rem;
    }

    .order-info-row {
        font-size: 0.78rem;
    }

    .order-card-actions .btn {
        font-size: 0.7rem;
        padding: 5px 6px;
    }
}

/* ========================================= */
/* ===== دسکتاپ: مخفی کردن کارت‌ها ===== */
/* ========================================= */
@media (min-width: 768px) {
    .order-cards {
        display: none;
    }
}
</style>