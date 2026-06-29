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
                    <!-- انتخاب آدرس -->
                    <div class="col-12" v-if="addresses.length">
                        <label class="form-label">آدرس</label>
                        <Treeselect v-model="selectedAddress" :multiple="false" :options="addresses"
                            placeholder="انتخاب آدرس..." />
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
                            <input v-model.number="selectedQuantity" type="number" min="1" class="form-control "
                                placeholder="تعداد" />
                            <button type="button" class="btn btn-success " @click="addProduct">افزودن</button>
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

                    <!-- روش حمل و نقل -->
                    <div v-if="form.items.length" class="col-12">
                        <label class="form-label">روش حمل و نقل</label>
                        <Treeselect :normalizer="shippingNormalizer" v-if="shippings.length" :valueFormat="'object'"
                            v-model="form.shipping_method" :multiple="false" :options="shippings"
                            placeholder="انتخاب روش حمل..." />
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
                        <p>هزینه حمل: <strong>{{ shippingCost.toLocaleString() }} تومان</strong></p>
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
    address_id: null,
    shipping_method: null,
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
let selectedAddress = ref(null);
const addresses = ref([])
const selectedProduct = ref(null)
const selectedQuantity = ref(1)
const shippingCost = ref(0)
const loading = ref(false)
let userOptions = ref([]);
let productOptions = ref([]);
let shippings = ref([]);
let selectedUser = ref(null);
let wallet = ref(null);
watch(() => selectedUser.value, (newUser) => {
    if (newUser) {
        fetchAddresses(newUser.addresses);
        wallet.value = newUser.wallet;
    } else {
        addresses.value = [];
        wallet.value = null;
    }
})
watch(() => selectedAddress.value, (newAddress) => {
    if (newAddress) {
        showAvalibleShipping()
    } else {
        shippings.value = [];
    }
})
// محصولات انتخاب شده → محاسبه جمع
const subtotal = computed(() =>
    form.value.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
)

const total = computed(() => subtotal.value + shippingCost.value - discount_amount.value)
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
    userOptions.value = data.data.map(u => ({ id: u.id, label: u.full_name, addresses: u.addresses, wallet: u.wallet }));
}
async function showAvalibleShipping() {
    shippings.value = [];
    form.value.shipping_method = null;
    const { data } = await axios.get(`shippings/avalible-shipping`, {
        params: {
            addressId: selectedAddress.value,
            subTotal: subTotal.value,
            quantity: sumQuantity.value,
        }
    });
    shippings.value = data.data
}
// گرفتن آدرس‌های کاربر انتخاب شده
const fetchAddresses = async (asses) => {
    addresses.value = asses.map(a => ({ id: a.id, label: `${a.receiver_name} - ${a.address_line} - ${a.phone} ` }))
}

const shippingNormalizer = (node) => {
    return {
        id: node.method_id,
        label: node.shipping_method,
    }
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
    productOptions.value = await convertToSelectableProduct(data.data);
}

async function convertToSelectableProduct(productList) {
    let finalList = [];
    productList.forEach(product => {
        // اگر بیشتر از یک تنوع پایه داشته باشه
        if (product.variants.length > 1) {
            product.variants.forEach((variant) => {
                let obj = {
                    id: variant.id,
                    product_id: product.id,
                    price: variant.price,
                    title: `${variant.id} - ${product.title} || ${variant.values.map((att) => { return att.value }).join("-")} || موجودی :${variant.stock}`,
                    isDisabled: variant.stock > 0 ? false : true
                }
                finalList.push(obj);
            });
        } else {
            let obj = {};
            obj.isDisabled = product.variants[0].stock > 0 ? false : true;
            obj.id = product.variants[0].id;
            obj.title = product.variants[0].id + " - " + product.title;
            obj.price = product.price;
            obj.product_id = product.id;
            finalList.push(obj);
        }
    });

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
    showAvalibleShipping()
}

// حذف محصول
const removeProduct = (index) => {
    form.value.items.splice(index, 1)
}
// وقتی روش حمل انتخاب شد → هزینه اضافه شود
watch(() => form.value.shipping_method, async (val) => {
    if (!val || !selectedAddress.value) {
        return
    }
    shippingCost.value = val.cost
})
// ثبت سفارش
const submitOrder = async () => {
    if (!selectedUser.value || !selectedAddress.value || !form.value.items.length || !form.value.shipping_method) {
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
        formData.append("address_id", selectedAddress.value)
        formData.append("shipping_method_id", form.value.shipping_method?.method_id)
        formData.append("subtotal", subtotal.value)
        formData.append("discount_amount", discount_amount.value)
        formData.append("shipping_cost", shippingCost.value)
        formData.append("total", total.value)
        form.value.items.forEach((item, index) => {
            formData.append(`items[${index}][product_id]`, item.product_id);
            formData.append(`items[${index}][product_variant_id]`, item.id);
            formData.append(`items[${index}][quantity]`, item.quantity);
            formData.append(`items[${index}][price]`, item.price);
        })
        await axios.post('/orders-create-by-admin', formData)
        toast.success('سفارش با موفقیت ثبت شد')
        form.value = { user_id: null, address_id: null, shipping_method: null, items: [] }
        addresses.value = []
        shippingCost.value = 0;
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