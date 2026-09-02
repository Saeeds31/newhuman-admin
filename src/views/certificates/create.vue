<template>
    <div class="container mt-4 bg-gray" v-if="checkPermission(['certificate_store'])">
        <h3 class="p-2">
            <i class="bi bi-award"></i>
            <span>ایجاد گواهینامه جدید</span>
        </h3>
        <b-form @submit.prevent="submitForm">
            <b-row>
                <!-- User -->
                <b-col cols="12" md="6">
                    <label class="form-label">انتخاب کاربر</label>

                    <multiselect @search-change="loadUsers" v-model="selectedUser" placeholder="انتخاب کاربر"
                        open-direction="bottom" :options="userOptions" label="label" track-by="id" :searchable="true"
                        :multiple="false" :close-on-select="true" :show-labels="false">
                        <template slot="noOptions">
                            جستجو کنید
                        </template>
                        <template slot="noResult">
                            <span v-if="isRequesting" v-text="'در حال جستجو...'" />
                            <span v-else v-text="'موردی یافت نشد'"></span>
                        </template>
                    </multiselect>
                </b-col>

                <!-- Product -->
                <b-col cols="12" md="6">
                    <label class="form-label">افزودن محصول</label>
                    <div class="gap-2 align-items-center selectProduct">
                        <multiselect @search-change="loadProducts" v-model="selectedProduct" placeholder="انتخاب محصول"
                            open-direction="bottom" :options="productOptions" label="title" track-by="id"
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
                </b-col>

                <!-- Number -->
                <b-col cols="12" md="6">
                    <b-form-group label="شماره گواهینامه" label-for="number">
                        <b-form-input id="number" v-model="form.number" placeholder="CERT-2026-001" />
                        <small class="text-danger" v-if="errors.number">{{ errors.number[0] }}</small>
                    </b-form-group>
                </b-col>

                <!-- Date Acquisition -->
                <b-col cols="12" md="6">
                    <b-form-group label="تاریخ دریافت" label-for="date_acquisition">
                        <date-picker display-format="jYYYY/jMM/jDD" format="YYYY-MM-DD"
                            v-model="form.date_acquisition"></date-picker>
                        <small class="text-danger" v-if="errors.date_acquisition">{{ errors.date_acquisition[0]
                        }}</small>
                    </b-form-group>
                </b-col>

                <!-- Image -->
                <b-col cols="12" md="6">
                    <b-form-group label="تصویر گواهینامه">
                        <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".jpg,.png,.webp,.jpeg" theme="grid"
                            deletable sortable />
                        <small class="text-danger" v-if="errors.image">{{ errors.image[0] }}</small>
                    </b-form-group>
                </b-col>

                <!-- File -->
                <b-col cols="12" md="6">
                    <b-form-group label="فایل گواهینامه (PDF)">
                        <VueFileAgent @select="fileLoaded" :maxFiles="1" accept=".pdf" theme="grid" deletable
                            sortable />
                        <small class="text-danger" v-if="errors.file">{{ errors.file[0] }}</small>
                    </b-form-group>
                </b-col>

                <!-- Description -->
                <b-col cols="12">
                    <b-form-group label="توضیحات">
                        <b-form-textarea v-model="form.description" rows="3" placeholder="توضیحات گواهینامه" />
                        <small class="text-danger" v-if="errors.description">{{ errors.description[0] }}</small>
                    </b-form-group>
                </b-col>
            </b-row>

            <div class="mt-3">
                <b-button type="submit" :disabled="loading" variant="primary">
                    <i class="bi bi-save2"> ذخیره </i>
                </b-button>
                <router-link to="/certificates" class="btn btn-secondary mx-2">
                    <i class="bi bi-x-circle"> انصراف </i>
                </router-link>
            </div>
        </b-form>
    </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { BForm, BFormGroup, BFormInput, BFormTextarea, BButton, BRow, BCol } from 'bootstrap-vue-3'
import { displayError } from '@/composable/useError'
import { useAdmin } from '@/stores/modules/admin'

const store = useAdmin();
const checkPermission = store.checkPermission;
const loading = ref(false);

const form = reactive({

    number: '',
    date_acquisition: '',
    description: '',
    image: '',
    file: ''
})

const errors = reactive({})
const users = ref([])
const products = ref([])

function imageLoaded(files) {
    form.image = files[0]?.file || ''
}

function fileLoaded(files) {
    form.file = files[0]?.file || ''
}
let abortController = null;
let userOptions = ref([]);
let selectedUser = ref();
const loadUsers = async (searchQuery) => {
    if (abortController) {
        abortController.abort();
    }
    abortController = new AbortController();
    try {
        const { data } = await axios.get('/users?search=' + searchQuery ?? '', {
            signal: abortController.signal,
        })
        const ops = data.data.map(u => ({ id: u.id, label: `${u.full_name} (${u.mobile})` }))
        userOptions.value = ops;

    } catch (error) {
        if (axios.isCancel(error)) {
            console.log('درخواست قبلی کنسل شد:', error.message);
        } else {
            toast.error('خطا در جستجوی کاربران')
        }
    }
}
let abortController1 = null;
let productOptions = ref([]);
let selectedProduct = ref();
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
const submitForm = async () => {
    Object.keys(errors).forEach(key => delete errors[key])
    loading.value = true

    try {
        const formData = new FormData()
        for (const key in form) {
            if (form[key]) {
                formData.append(key, form[key])
            }
        }
        formData.append("user_id", selectedUser.value?.id || '')
        formData.append("product_id", selectedProduct.value?.id || '')
        await axios.post('/certificates', formData, {
            headers: { 'Content-Type': 'multipart/form-data' }
        })

        toast.success('گواهینامه با موفقیت ذخیره شد ✅')

        // Reset form
        Object.keys(form).forEach(key => {
            if (key !== 'image' && key !== 'file') {
                form[key] = ''
            }
        })
        form.image = ''
        form.file = ''

    } catch (err) {
        if (err.response && err.response.status === 422) {
            Object.assign(errors, err.response.data.errors)
            displayError(err.response.data.errors)
        } else if (err.response && err.response.status === 409) {
            toast.error(err.response.data.message)
        } else {
            toast.error(err.response.data.message)
        }
    } finally {
        loading.value = false
    }
}
</script>