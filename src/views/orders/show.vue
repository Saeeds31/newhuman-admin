<template>
    <b-container fluid class="py-4" v-if="checkPermission(['order_view'])">
        <b-row>
            <!-- ستون جزئیات سفارش -->
            <b-col cols="12" lg="4">
                <b-card header="جزئیات سفارش" class="mb-3">
                    <ul class="list-unstyled mb-0">
                        <li><strong>مشتری:</strong> {{ order.user?.full_name }}</li>
                        <template v-if="order.address">
                            <li><strong>گیرنده:</strong> {{ order.address.receiver_name }}</li>
                            <li><strong>آدرس:</strong> {{
                                order.address.province.name + " - " + order.address.city.name + " - "
                                + order.address.address_line
                            }}</li>
                            <li><strong>کدپستی:</strong> {{ order.address.postal_code }}</li>
                            <li><strong>شماره تماس گیرنده:</strong> {{ order.address.phone }}</li>
                        </template>
                        <li><strong>روش حمل:</strong> {{ order.shipping_method?.name }}</li>
                        <li><strong>روش پرداخت:</strong> {{ paymentMethods[order.payment_method] }}</li>
                    </ul>
                </b-card>
            </b-col>
            <!-- ستون خلاصه سفارش -->
            <b-col cols="12" lg="4">
                <b-card header="خلاصه سفارش">
                    <ul class="list-unstyled mb-0">
                        <li><strong>جمع جزء:</strong> {{ formatPrice(order.subtotal) }}</li>
                        <li><strong>تخفیف:</strong> {{ formatPrice(order.discount_amount) }}</li>
                        <li><strong>هزینه ارسال:</strong> {{ formatPrice(order.shipping_cost) }}</li>
                        <li class="mt-2">
                            <strong class="h5">جمع کل:</strong>
                            <span class="h5">{{ formatPrice(order.total) }}</span>
                        </li>
                    </ul>
                </b-card>


            </b-col>
            <b-col cols="12" lg="4">
                <b-card header="وضعیت‌ها" class="mb-3">
                    <b-form-group label="وضعیت سفارش" label-for="order-status">
                        <b-form-select id="order-status" v-model="order.status" :options="orderStatusOptions" />
                    </b-form-group>

                    <b-form-group label="وضعیت پرداخت" label-for="payment-status">
                        <b-form-select id="payment-status" disabled v-model="order.payment_status"
                            :options="paymentStatusOptions" />
                    </b-form-group>

                    <b-button variant="primary" block class="mt-3" @click="updateOrder">
                        ذخیره تغییرات
                    </b-button>
                </b-card>

            </b-col>
        </b-row>
        <b-row>
            <!-- ستون آیتم‌های سفارش -->
            <b-col cols="12" lg="12">
                <b-card header="آیتم‌های سفارش">
                    <div class="card-body table-responsive">
                        <table class="table table-bordered align-middle text-center">
                            <thead>
                                <tr>
                                    <th>#</th>
                                    <th>محصول</th>
                                    <th>تصویر</th>
                                    <th>تعداد</th>
                                    <th>قیمت واحد(تومان)</th>
                                    <th>مجموع(تومان)</th>
                                </tr>
                            </thead>
                            <tbody>
                                <tr v-for="(item, index) in order.items" :key="item.id">
                                    <td>{{ index + 1 }}</td>
                                    <td>{{ getProductTitle(item) }}</td>
                                    <td>{{ getProductImage(item) }}</td>
                                    <td>{{ item.quantity }}</td>
                                    <td>{{ item.price.toLocaleString() }}</td>
                                    <td>{{ Number(item.quantity * item.price).toLocaleString() }}</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>

                </b-card>
            </b-col>
        </b-row>
    </b-container>
</template>

<script setup>
import { ref, onMounted } from "vue"
import axios from "axios"
import { toast } from "vue3-toastify"
import "vue3-toastify/dist/index.css"
import { useRoute } from "vue-router"

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute();
const order = ref({ items: [] })

const itemFields = [

]

const orderStatusOptions = [
    { value: "pending", text: "در انتظار" },
    { value: "reserved", text: "رزرو شده" },
    { value: "processing", text: "در حال پردازش" },
    { value: "shipped", text: "ارسال شده" },
    { value: "completed", text: "تکمیل شده" },
    { value: "canceled", text: "لغو شده" },
    { value: "returned", text: "مرجوع شده" },
]

const paymentMethods = {
    online: "پرداخت آنلاین",
    wallet: "کیف پول",
    cod: "پرداخت در محل",
}

const paymentStatusOptions = [
    { value: "pending", text: "در انتظار پرداخت" },
    { value: "paid", text: "پرداخت شده" },
    { value: "failed", text: "ناموفق" },
    { value: "refunded", text: "برگشت داده شده" },
]

const fetchOrder = async () => {
    try {
        const { data } = await axios.get(`/orders/${route.params.id}`)
        order.value = data.data
    } catch (e) {
        toast.error("خطا در گرفتن اطلاعات سفارش")
    }
}

const updateOrder = async () => {
    try {
        let fd = new FormData();
        fd.append("status", order.value.status)
        await axios.post(`/orders-change-status/${order.value.id}`, fd)
        toast.success("سفارش با موفقیت بروزرسانی شد")
    } catch (e) {
        toast.error("خطا در بروزرسانی سفارش")
    }
}

onMounted(fetchOrder)
function getProductTitle(item) {
    let title = "";
    title = item.product.title;
    if (item.variant && item.variant.values.length) {
        title += ` [${item.variant.values.map(att => att.value).join("-")}]`
    }
    return title;
}
function getProductImage(item) {
    if (!item.main_image) {
        return ""
    }
    return baseImageAddress + item.main_image
}
const formatPrice = (val) => {
    if (!val) return "0"
    return new Intl.NumberFormat("fa-IR").format(val) + " تومان"
}
</script>

<style scoped>
ul li {
    margin-bottom: 0.5rem;
}
</style>