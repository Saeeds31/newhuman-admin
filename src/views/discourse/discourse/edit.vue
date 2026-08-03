<template>
  <div v-if="checkPermission(['discourse_update'])" class="container py-4">
    <b-card>
      <h5 class="mb-3">ویرایش گفتومان</h5>
      <b-form @submit.prevent="handleSubmit">
        <b-row>
          <!-- Title -->
          <b-col cols="12" md="6">
            <b-form-group label="عنوان" label-for="title">
              <b-form-input id="title" v-model="form.title" />
              <small v-if="errors.title" class="text-danger">{{ errors.title[0] }}</small>
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
              <b-form-input id="slug" v-model="form.slug" />
              <small v-if="errors.slug" class="text-danger">{{ errors.slug[0] }}</small>
            </b-form-group>
          </b-col>

          <b-col cols="12" md="6">
            <b-form-group label="دسته‌بندی " label-for="discourse_category_id">
              <Treeselect v-if="parentOptions.length" id="discourse_category_id" :multiple="false"
                v-model="form.discourse_category_id" :normalizer="normalizer" :options="parentOptions"
                placeholder="انتخاب دسته‌بندی " :clearable="true" />
              <small class="text-danger" v-if="errors.discourse_category_id">{{ errors.discourse_category_id[0]
              }}</small>
            </b-form-group>
          </b-col>
          <b-col cols="12" md="12">
            <b-form-group label="ادرس ویدیو" label-for="video">
              <b-form-input id="video" v-model="form.video" />
              <small class="text-danger" v-if="errors.video">{{ errors.video[0] }}</small>
            </b-form-group>
          </b-col>
          <b-col cols="12" md="12">
            <b-form-group label="تصویر" label-for="image">
              <VueFileAgent @update:raw-model-value="imageLoaded" :raw-model-value="oldImage" :maxFiles="1"
                accept=".pdf,.jpg,.png,.webp" theme="grid" deletable sortable>
              </VueFileAgent>

              <small v-if="errors.main_image" class="text-danger">{{ errors.main_image[0] }}</small>
            </b-form-group>
          </b-col>
          <b-col cols="12">
            <b-form-group label="توضیح کوتاه">
              <b-form-textarea v-model="form.short_description" rows="2" />
              <small class="text-danger" v-if="errors.short_description">{{ errors.short_description[0]
              }}</small>
            </b-form-group>
          </b-col>
    

          <!-- Description (Editor) -->
          <b-col cols="12">
            <b-form-group label="توضیح کامل" label-for="description">
              <Editor v-model="form.description" />
              <small v-if="errors.description" class="text-danger">{{ errors.description[0] }}</small>
            </b-form-group>
          </b-col>



        </b-row>

        <div class="mt-3">
          <b-button type="submit" :disabled="loading" variant="primary">
            <i class="bi bi-save2"></i>
            <span class="mx-2">
              ویرایش
              گفتومان
            </span>
          </b-button>
        </div>
      </b-form>
    </b-card>
  </div>
</template>

<script setup>
import { reactive, ref, onMounted } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { BForm, BFormGroup, BFormInput, BButton, BCard, BRow, BCol } from 'bootstrap-vue-3'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import Editor from '@/components/shared/editor.vue';
import { useRoute } from 'vue-router'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
let loading = ref(false);
const route = useRoute();
const oldImage = ref([]);
const form = reactive({
  title: '',
  slug: '',
  main_image: [],
  short_description: '',
  description: '',
  video: '',
  discourse_with: '',
  discourse_category_id: []

})
const normalizer = (node) => {
  // تبدیل کلیدها به فرمت استاندارد کامپوننت
  return {
    id: node.id,
    label: node.title,
    children: node.children
  }
}
const errors = reactive({})
const parentOptions = ref([])
function getParentOption() {
  axios.get("/discourse-categories").then((res) => {
    parentOptions.value = res.data.data
  })
}
onMounted(async () => {
  loading.value = true;
  try {
    // GET اطلاعات مقاله
    const res = await axios.get(`/discourse/${route.params.id}`)
    if (res.data.main_image)
      oldImage.value =
        [{
          name: res.data.main_image.split('/').pop(),
          size: 0,
          type: 'image/jpeg',
          ext: res.data.main_image.split('.').pop(),
          url: `${baseImageAddress}${res.data.main_image}`,
        }];


    Object.assign(form, res.data);
    getParentOption()
  } catch (err) {
    console.log(err);
    toast.error('خطا در دریافت اطلاعات گفتومان ❌')
  } finally {
    loading.value = false;
  }
})
function imageLoaded(files) {
  if (files.length) {
    form.main_image = files[0].file
  } else {
    form.main_image = '';
  }
}
const handleSubmit = async () => {
  Object.keys(errors).forEach(k => delete errors[k])
  try {
    const formData = new FormData()
    for (const key in form) {
      if (key != 'main_image') formData.append(key, form[key])
    }
    formData.append("_method", "PUT");

    if (form.main_image) {
      formData.append("main_image", form.main_image);
    }
    await axios.post(`/discourse/${route.params.id}`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    })
    toast.success('گفتومان با موفقیت ویرایش شد ✅');
  } catch (err) {
    if (err.response?.status === 422) {
      Object.assign(errors, err.response.data.errors)
      toast.error('خطاهای فرم را بررسی کنید ❌')
    } else {
      toast.error('خطا در ارسال اطلاعات ❌')
    }
  }
}
</script>
