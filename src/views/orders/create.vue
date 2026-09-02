<template>
    <div class="container mt-4" v-if="checkPermission(['order_store'])">
        <div class="row">
            <!-- ستون اصلی -->
            <div class="col-md-8 bg-gray">
                <h3 class=" p-2">
                    <i class="bi bi-plus"></i>
                    <span>
                        ثبت سفارش جدید
                    </span>
                </h3>
                <form @submit.prevent="submitOrder" class="row g-3">
                    <!-- انتخاب کاربر -->
                    <div class="col-md-12">
                        <label class="form-label">انتخاب کاربر</label>
                        <multiselect @search-change="loadUsers" v-model="selectedUser" placeholder="انتخاب کاربر"
                            open-direction="bottom" :options="userOptions" label="label" track-by="id"
                            :searchable="true" :multiple="false" :close-on-select="true" :show-labels="false">
                            <template slot="noOptions">
                                جستجو کنید
                            </template>
                            <template slot="noResult">
                                <span v-if="isRequesting" v-text="'در حال جستجو...'" />
                                <span v-else v-text="'موردی یافت نشد'"></span>
                            </template>
                        </multiselect>
                    </div>


                    <!-- افزودن محصول -->
                    <div class="col-12">
                        <label class="form-label">افزودن محصول</label>
                        <div class="gap-2 align-items-center selectProduct">
                            <multiselect @search-change="loadProducts" v-model="selectedProduct"
                                placeholder="انتخاب محصول" open-direction="bottom" :options="productOptions"
                                label="title" track-by="id" :searchable="true" :multiple="false" :close-on-select="true"
                                :show-labels="false">
                                <template slot="noOptions">
                                    جستجو کنید
                                </template>
                                <template slot="noResult">
                                    <span v-if="isRequesting" v-text="'در حال جستجو...'" />
                                    <span v-else v-text="'موردی یافت نشد'"></span>
                                </template>
                            </multiselect>
                            <button v-if="!form.items.length" type="button" class="btn btn-success "
                                @click="addProduct">افزودن</button>
                        </div>
                    </div>

                    <!-- لیست محصولات انتخاب‌شده -->
                    <div class="col-12" v-if="form.items.length">
                        <h5>محصولات انتخاب شده</h5>
                        <ul class="list-group">
                            <li v-for="(item, index) in form.items" :key="index"
                                class="list-group-item d-flex justify-content-between align-items-center">
                                {{ item.title }} × {{ item.quantity }}
                                <span>
                                    {{ (item.price * item.quantity).toLocaleString() }} تومان
                                    <button type="button" class="btn btn-sm btn-danger ms-2"
                                        @click="removeProduct(index)">حذف</button>
                                </span>
                            </li>
                        </ul>
                    </div>


                </form>
            </div>

            <!-- ستون جمع سفارش -->
            <div class="col-md-4 ">
                <div class="card">
                    <div class="card-header">
                        <h3>
                            <span>
                                جمع سفارش
                            </span>
                        </h3>
                    </div>
                    <div class="card-body">

                        <p v-if="wallet">موجودی کیف پول: <strong>{{ wallet.balance.toLocaleString() }} تومان</strong>
                        </p>
                        <p>جمع محصولات: <strong>{{ subtotal.toLocaleString() }} تومان</strong></p>
                        <p>تخفیف: <input type="number" class="form-control" v-model="discount_amount"></p>

                        <hr />
                        <h5>مبلغ نهایی: <strong>{{ total.toLocaleString() }} تومان</strong></h5>
                    </div>
                    <div class="card-footer">
                        <button class="btn btn-primary w-100" @click="submitOrder" :disabled="loading">
                            {{ loading ? 'در حال ثبت...' : 'ثبت سفارش' }}
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, computed, watch } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const form = ref({
    user_id: null,
    items: []
})
let sumQuantity = computed(() => {
    return form.value.items.reduce((accumulator, item) => {
        return accumulator + item.quantity
    }, 0);
})
let subTotal = computed(() => {
    return form.value.items.reduce((accumulator, item) => {
        return accumulator + (item.price * item.quantity)
    }, 0);
})

let discount_amount = ref(0);
const selectedProduct = ref(null)
const selectedQuantity = ref(1)
const loading = ref(false)
let userOptions = ref([]);
let productOptions = ref([]);
let selectedUser = ref(null);
let wallet = ref(null);

// محصولات انتخاب شده → محاسبه جمع
const subtotal = computed(() =>
    form.value.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
)

const total = computed(() => subtotal.value - discount_amount.value)
let abortController = null;
// لود کاربران
const loadUsers = async (search) => {
    if (abortController) {
        abortController.abort();
    }

    abortController = new AbortController();

    const { data } = await axios.get('/users', {
        params: { search },
        signal: abortController.signal,

    })
    userOptions.value = data.data.map(u => ({ id: u.id, label: u.full_name, wallet: u.wallet }));
}



let abortController1 = null;

const loadProducts = async (search) => {
    if (!search && search.length < 2) return;
    if (abortController1) {
        abortController1.abort();
    }
    abortController1 = new AbortController();
    const { data } = await axios.get('/products', {
        params: { search },
        signal: abortController1.signal,
    })
    productOptions.value = await convertToSelectableProduct(data.data.data);
}

async function convertToSelectableProduct(productList) {
    console.log(productList);

    let finalList = [];
    productList.forEach(product => {
        let obj = {};
        obj.id = product.id;
        obj.title = product.title + " " + product.parent?.title || " ";
        obj.price = product.price;
        obj.product_id = product.id;
        finalList.push(obj);

    })
    return finalList;
}
// افزودن محصول به سفارش
const addProduct = () => {
    if (!selectedProduct.value || selectedQuantity.value < 1) {
        return toast.error('لطفاً محصول و تعداد را انتخاب کنید')
    }
    const product = selectedProduct.value
    let findedIndex = form.value.items.findIndex(item => item.id == product.id)
    if (findedIndex != -1) {
        form.value.items[findedIndex].quantity += selectedQuantity.value;
    } else {
        form.value.items.push({
            id: product.id,
            product_id: product.product_id,
            title: product.title,
            price: product.price,
            quantity: selectedQuantity.value
        })
    }
    selectedQuantity.value = 1;
}

// حذف محصول
const removeProduct = (index) => {
    form.value.items.splice(index, 1)
}

// ثبت سفارش
const submitOrder = async () => {
    if (!selectedUser.value || !form.value.items.length) {
        return toast.error('لطفاً همه فیلدها را پر کنید')
    }
    loading.value = true;
    let nullProduct = null;
    let items = form.value.items;
    for (let i = 0; i < items.length; i++) {
        if (isNaN(items[i].id)) {
            nullProduct = items[i];
            break;
        }
    }
    if (nullProduct) {
        loading.value = false
        return toast.error(`محصول ${nullProduct.title}دارای تنوع است لطفا یکی از تنوع های آن را انتخاب کنید`)
    }
    try {
        let formData = new FormData();
        formData.append("user_id", selectedUser.value.id)
        formData.append("subtotal", subtotal.value)
        formData.append("discount_amount", discount_amount.value)
        formData.append("total", total.value)
        form.value.items.forEach((item, index) => {
            formData.append(`product_id`, item.product_id);
        })
        await axios.post('/orders-create-by-admin', formData)
        toast.success('سفارش با موفقیت ثبت شد')
        form.value = { user_id: null, items: [] }
        wallet.value = null;
        selectedUser.value = null;
    } catch (e) {
        console.log(e);
        toast.error(e.response.data.message)
    } finally {
        loading.value = false
    }
}
</script>

<style scoped>
.card {
    border-radius: 10px;
    box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
}

.selectProduct {
    display: grid;
    grid-template-columns: 10fr 1fr 2fr;
}
</style>