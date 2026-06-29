<template>
  <div class="container py-4 bg-gray mt-4" v-if="checkPermission(['story_update'])">
    <h3 class=" p-2">
      <i class="bi bi-list-columns-reverse"></i>
      <span>ویرایش استوری
      </span>
    </h3>
    <b-form @submit.prevent="handleSubmit">
      <b-row>
        <b-col cols="12" md="6">
          <b-form-group label="عنوان" label-for="title">
            <b-form-input id="title" v-model="form.title" />
            <small v-if="errors.title" class="text-danger">{{ errors.title[0] }}</small>
          </b-form-group>
        </b-col>

        <!-- Link -->
        <b-col cols="12" md="6">
          <b-form-group label="لینک" label-for="link">
            <b-form-input id="link" v-model="form.link" />
            <small v-if="errors.link" class="text-danger">{{ errors.link[0] }}</small>
          </b-form-group>
        </b-col>

        <div class="col-md-12 mb-3">
          <label class="form-label">وضعیت</label>
          <select v-model="form.status" class="form-select">
            <option value="">انتخاب کنید</option>
            <option value="draft">پیشنویس</option>
            <option value="published">انتشار</option>
            <option value="archived">آرشیو</option>
          </select>
          <span v-if="errors.status" class="text-danger">{{ errors.status[0] }}</span>
        </div>

        <b-col cols="12" md="6">
          <b-form-group label="کاور" label-for="icon">
            <VueFileAgent @update:raw-model-value="imageLoaded" :raw-model-value="oldImage" :maxFiles="1"
              accept=".pdf,.jpg,.png,.webp" theme="grid" deletable sortable>
            </VueFileAgent>
            <small v-if="errors.cover" class="text-danger">{{ errors.cover[0] }}</small>
          </b-form-group>
        </b-col>

        <b-col cols="12" md="6">
          <b-form-group label="ویدیو" label-for="icon">
            <VueFileAgent @update:raw-model-value="imageLoaded1" :raw-model-value="oldVideo" :maxFiles="1"
              accept=".mp4,.mkv" theme="grid" deletable sortable>
            </VueFileAgent>
            <small v-if="errors.video" class="text-danger">{{ errors.video[0] }}</small>
          </b-form-group>
        </b-col>

      </b-row>

      <div class="mt-3">
        <b-button type="submit" :disabled="loading" variant="primary">
          <i class="bi bi-save2"></i>
          <span class="mx-2">
            ویرایش استوری
          </span>
        </b-button>
      </div>
    </b-form>
  </div>
</template>

<script setup>
import { reactive, onMounted, ref } from 'vue'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import Treeselect from 'vue3-treeselect'
// import the styles
import 'vue3-treeselect/dist/vue3-treeselect.css'

import { BForm, BFormGroup, BFormInput, BButton, BCard, BRow, BCol } from 'bootstrap-vue-3'
import { useRoute } from 'vue-router'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute()

let loading = ref(false);
const form = reactive({
  title: '',
  link: '',
  parent_id: null,
  cover: '',
  video: '',
})

const errors = reactive({})
let storyOptions = ref([])
let oldImage = ref([])
let oldVideo = ref([])

onMounted(async () => {
  try {
    const storyRes = await axios.get(`/stories/${route.params.id}`)
    Object.assign(form, storyRes.data.data)
    if (storyRes.data.data.cover)
      oldImage.value =
        [{
          name: storyRes.data.data.cover.split('/').pop(),
          size: 0,
          type: 'image/jpeg',
          ext: storyRes.data.data.cover.split('.').pop(),
          url: `${baseImageAddress}${storyRes.data.data.cover}`,
        }];
    if (storyRes.data.data.video)
      oldVideo.value =
        [{
          name: storyRes.data.data.video.split('/').pop(),
          size: 0,
          type: 'video/mp4',
          ext: storyRes.data.data.video.split('.').pop(),
          url: `${baseImageAddress}${storyRes.data.data.video}`,
        }];
  } catch {
    toast.error('خطا در دریافت اطلاعات استوری ❌')
  }
})

function imageLoaded(files) {
  if (files.length) {
    form.cover = files[0].file
  } else {
    form.cover = '';
  }
}
function imageLoaded1(files) {
  if (files.length) {
    form.video = files[0].file
  } else {
    form.video = '';
  }
}
const normalizer = (node) => {
  // تبدیل کلیدها به فرمت استاندارد کامپوننت
  return {
    id: node.id,
    label: node.title,
    children: node.children
  }
}
const handleSubmit = async () => {
  Object.keys(errors).forEach(k => delete errors[k])
  loading.value = true;
  try {
    const formData = new FormData()
    for (const key in form) {
      if (key != 'cover' && key != 'video') formData.append(key, form[key])
    }
    if (form.cover) {
      formData.append("cover", form.cover)
    }
    if (form.video) {
      formData.append("video", form.video)
    }
    formData.append("_method", "PUT")
    await axios.post(`/stories/${route.params.id}`, formData)
    toast.success('استوری با موفقیت ویرایش شد ✅')
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