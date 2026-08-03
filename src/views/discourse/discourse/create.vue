<template>
    <div class="container mt-4 bg-gray" v-if="checkPermission(['article_store'])">
        <h3 class=" p-2">
            <i class="bi bi-book-half"></i>
            <span>
                ایجاد گفتومان جدید
            </span>
        </h3>
        <b-form @submit.prevent="submitForm">
            <b-row>
                <!-- Title -->
                <b-col cols="12" md="6">
                    <b-form-group label="عنوان" label-for="title">
                        <b-form-input id="title" v-model="form.title" placeholder="عنوان گفتومان" />
                        <small class="text-danger" v-if="errors.title">{{ errors.title[0] }}</small>
                    </b-form-group>
                </b-col>
                <b-col cols="12" md="6">
                    <b-form-group label="گفتگو با" label-for="discourse_with">
                        <b-form-input id="discourse_with" v-model="form.discourse_with" placeholder="عنوان گفتومان" />
                        <small class="text-danger" v-if="errors.discourse_with">{{ errors.discourse_with[0] }}</small>
                    </b-form-group>
                </b-col>
                <!-- Slug -->
                <b-col cols="12" md="6">
                    <b-form-group label="Slug" label-for="slug">
                        <b-form-input id="slug" v-model="form.slug" placeholder="slug گفتومان" />
                        <small class="text-danger" v-if="errors.slug">{{ errors.slug[0] }}</small>
                    </b-form-group>
                </b-col>
                <b-col cols="12" md="6">
                    <b-form-group label="دسته‌بندی " label-for="discourse_category_id">
                        <Treeselect id="discourse_category_id" :multiple="false" v-model="form.discourse_category_id"
                            :normalizer="normalizer" :options="parentOptions" placeholder="انتخاب دسته‌بندی "
                            :clearable="true" :valueConsistsOf="'ALL'" />
                        <small class="text-danger" v-if="errors.discourse_category_id">{{
                            errors.discourse_category_id[0]
                        }}</small>
                    </b-form-group>
                </b-col>

                <b-col cols="12" md="12">
                    <b-form-group label="ادرس ویدیو" label-for="video">
                        <b-form-input id="video" v-model="form.video"  />
                        <small class="text-danger" v-if="errors.video">{{ errors.video[0] }}</small>
                    </b-form-group>
                </b-col>
                <!-- Image -->
                <b-col cols="12" md="12">
                    <b-form-group label="تصویر (URL)">
                        <VueFileAgent @select="imageLoaded" :maxFiles="1" accept=".pdf,.jpg,.png,.webp" theme="grid"
                            deletable sortable />
                        <small class="text-danger" v-if="errors.main_image">{{ errors.main_image[0] }}</small>
                    </b-form-group>
                </b-col>
                <!-- Short Description -->
                <b-col cols="12">
                    <b-form-group label="توضیح کوتاه">
                        <b-form-textarea v-model="form.short_description" rows="2" />
                        <small class="text-danger" v-if="errors.short_description">{{ errors.short_description[0]
                        }}</small>
                    </b-form-group>
                </b-col>

                <!-- Description (CKEditor) -->
                <b-col cols="12">
                    <b-form-group label="توضیحات کامل">
                        <Editor v-model="form.description" />
                        <small class="text-danger" v-if="errors.description">{{ errors.description[0]
                        }}</small>
                    </b-form-group>
                </b-col>

            </b-row>

            <div class="mt-3">
                <b-button type="submit" :disabled="loading" variant="primary">
                    <i class="bi bi-save2">
                        ذخیره
                    </i>
                </b-button>
            </div>
        </b-form>
    </div>
</template>

<script setup>
import { reactive, ref } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { BForm, BFormGroup, BFormInput, BFormTextarea, BButton, BCard, BRow, BCol, BFormInvalidFeedback } from 'bootstrap-vue-3'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import Editor from '@/components/shared/editor.vue';
import { displayError } from '@/composable/useError'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
let loading = ref(false);
const form = reactive({
    title: '',
    video: '',
    slug: '',
    discourse_with: '',
    short_description: '',
    description: '',
    main_image: '',
    discourse_category_id: ''
})
const errors = reactive({})
function imageLoaded(files) {
    form.main_image = files[0].file
}
const normalizer = (node) => {
    // تبدیل کلیدها به فرمت استاندارد کامپوننت
    return {
        id: node.id,
        label: node.title,
        children: node.children
    }
}
const parentOptions = ref([])
function getParentOption() {
    axios.get("/discourse-categories").then((res) => {
        parentOptions.value = res.data.data
    })
}
getParentOption()
const submitForm = async () => {

    Object.keys(errors).forEach(key => delete errors[key])
    loading.value = true;
    try {
        const formData = new FormData()
        for (const key in form) {
            formData.append(key, form[key] ?? '')
        }

        await axios.post('/discourse', formData)
        toast.success('گفتومان با موفقیت ذخیره شد ✅')
        Object.keys(form).forEach(key => (form[key] = key))
    } catch (err) {
        if (err.response && err.response.status === 422) {
            Object.assign(errors, err.response.data.errors)
            displayError(err.response.data.errors)
        } else {
            toast.error('خطا در ارسال اطلاعات ❌')
        }
    } finally {
        loading.value = false;
    }
}
</script>