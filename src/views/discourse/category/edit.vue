<template>
  <div class="container py-4 bg-gray mt-3" v-if="checkPermission(['discourse_update'])">
    <h3 class=" p-2">
      <i class="bi bi-list-task"></i>
      <span>
        ویرایش دسته بندی
      </span>
    </h3><b-form @submit.prevent="handleSubmit">
      <b-row>
        <!-- Title -->
        <b-col cols="12" md="6">
          <b-form-group label="عنوان" label-for="title">
            <b-form-input id="title" v-model="form.title" />
            <small v-if="errors.title" class="text-danger">{{ errors.title[0] }}</small>
          </b-form-group>
        </b-col>

        <!-- Slug -->
        <b-col cols="12" md="6">
          <b-form-group label="Slug" label-for="slug">
            <b-form-input id="slug" v-model="form.slug" />
            <small v-if="errors.slug" class="text-danger">{{ errors.slug[0] }}</small>
          </b-form-group>
        </b-col>
        <!-- Meta Title -->
        <b-col cols="12" md="6">
          <b-form-group label="Meta Title" label-for="meta_title">
            <b-form-input id="meta_title" v-model="form.meta_title" />
            <small v-if="errors.meta_title" class="text-danger">{{ errors.meta_title[0] }}</small>
          </b-form-group>
        </b-col>

        <!-- Meta Description -->
        <b-col cols="12" md="6">
          <b-form-group label="Meta Description" label-for="meta_description">
            <b-form-input id="meta_description" v-model="form.meta_description" />
            <small v-if="errors.meta_description" class="text-danger">{{ errors.meta_description[0] }}</small>
          </b-form-group>
        </b-col>

        <!-- Description (Editor) -->
        <b-col cols="12">
          <b-form-group label="توضیح" label-for="description">
            <b-form-textarea id="description" v-model="form.description" rows="5"></b-form-textarea>
            <small v-if="errors.description" class="text-danger">{{ errors.description[0] }}</small>
          </b-form-group>
        </b-col>
      </b-row>

      <div class="mt-3">
        <b-button type="submit" :disabled="loading" variant="primary">
          <i class="bi bi-save2"></i>
          <span class="mx-2">
            ویرایش دسته‌بندی
          </span>
        </b-button>
      </div>
    </b-form>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { BForm, BFormGroup, BFormInput, BButton, BCard, BRow, BCol } from 'bootstrap-vue-3'
import Editor from '@/components/shared/Editor.vue'
import { useRoute } from 'vue-router'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute()
const form = reactive({
  title: '',
  slug: '',
  meta_title: '',
  meta_description: '',
  description: '',
})
let loading = ref(false);

const errors = reactive({})
const categories = ref([])

onMounted(async () => {
  try {
    // GET اطلاعات دسته‌بندی
    const res = await axios.get(`/discourse-categories/${route.params.id}`)
    Object.assign(form, res.data)
  } catch (err) {
    toast.error('خطا در دریافت اطلاعات دسته‌بندی ❌')
  }
})

const handleSubmit = async () => {
  Object.keys(errors).forEach(k => delete errors[k])
  loading.value = true;
  try {
    const formData = new FormData()
    for (const key in form) {
      formData.append(key, form[key] ?? null)
    }
    formData.append('_method', "PUT")
    await axios.post(`/discourse-categories/${route.params.id}`, formData)
    toast.success('دسته‌بندی با موفقیت ویرایش شد ✅')
  } catch (err) {
    if (err.response?.status === 422) {
      Object.assign(errors, err.response.data.errors)
      toast.error('خطاهای فرم را بررسی کنید ❌')
    } else {
      toast.error('خطا در ارسال اطلاعات ❌')
    }
  } finally {
    loading.value = false;
  }
}
</script>
