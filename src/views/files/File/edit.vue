<template>
  <div class="product-edit container py-4" v-if="checkPermission(['file_update'])">
    <div class="bg-gray">
      <h3>
        <i class="bi bi-info"></i>
        <span>
          ویرایش فایل
        </span>
      </h3>
      <form @submit.prevent="submitData">
        <div class="formSetp1 g-3">
          <div class=" border-box row">
            <!-- عنوان و قیمت -->
            <div class="col-md-6 mb-3">
              <label class="form-label">عنوان فایل</label>
              <input v-model="form.title" type="text" class="form-control" />
              <span v-if="errors.title" class="text-danger">{{ errors.title[0] }}</span>
            </div>

            <div class="col-md-6  mb-3">
              <label class="form-label">دسته‌بندی‌</label>
              <Treeselect v-if="form.category_id" v-model="form.category_id" :multiple="false"
                :options="categoryOptions" :normalizer="normalizer" />
              <span v-if="errors.category_id" class="text-danger">{{ errors.category_id[0] }}</span>
            </div>


            <div class="col-md-12 mb-3">
              <label class="form-label">فایل </label>
              <VueFileAgent @select="imageLoaded1" @update:rawModelValue="changeFile" :raw-model-value="oldFile"
                :maxFiles="1" accept=".jpg,.png,.webp" theme="grid" deletable sortable />
              <span v-if="errors.file" class="text-danger">{{ errors.file[0] }}</span>
            </div>
          </div>

        </div>


        <button type="submit" :disabled="loading" class="btn btn-primary mt-3">
          <i class="bi bi-save2"></i>
          <span>
            ویرایش فایل
          </span>
        </button>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, watch, onMounted } from 'vue'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import { toast } from 'vue3-toastify'
import Editor from '@/components/shared/editor.vue';
import axios from 'axios'
import { useRoute, useRouter } from 'vue-router'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
let route = useRoute();
let router = useRouter();
const fileId = route.params.id // آی‌دی محصول مورد ویرایش
const file = ref(null)

let loading = ref(false);

let mainFileDeleted = ref(false);
const form = ref({
  title: '', category_id: '', file: "",
})
let oldFile = ref([])
const errors = ref({})
const categoryOptions = ref([])
const normalizer = node => ({ id: node.id, label: node.title, children: node.children })

onMounted(async () => {
  await loadCategories()
  await loadFile()
})


async function loadCategories() {
  const res = await axios.get('/file-categories')
  categoryOptions.value = res.data.data.data
}

async function loadFile() {
  const res = await axios.get(`/files/${fileId}`)
  file.value = res.data.data
  Object.assign(form.value, file.value)
  if (file.value.file) {
    oldFile.value =
      [{
        name: file.value.file.split('/').pop(),
        size: 0,
        type: "/docs",
        ext: file.value.file.split('.').pop(),
        url: `${baseImageAddress}${file.value.file}`,
      }];
  }
}
function imageLoaded1(files) { form.value.file = files[0].file }



function changeFile() {
  if (!mainFileDeleted.value) {
    mainFileDeleted.value = true;
  }
}
async function submitData() {
  errors.value = {}
  loading.value = true;
  try {
    const formData = new FormData()
    Object.keys(form.value).forEach(key => {
      if (key == "file") {
        if (mainFileDeleted.value) {
          if (form.value.file && typeof form.value.file != 'string') {
            formData.append(key, form.value.file)
          }
        }
      }
      else if (form.value[key]) {
        formData.append(key, form.value[key] ?? "")
      }
    })
    formData.append("_method", "PUT")
    const res = await axios.post(`/files/${fileId}`, formData)
    file.value = res.data
    toast.success('فایل با موفقیت بروزرسانی شد!')
  } catch (e) {
    if (e.response?.data?.errors) errors.value.step1 = e.response.data.errors
    toast.error('خطا در ذخیره فایل')
  } finally {
    loading.value = false;
  }
}

</script>

<style scoped>
.step-buttons {
  flex-wrap: wrap;
}



@media (max-width:768px) {
  .step-buttons {
    flex-direction: column;
  }

  .step-divider {
    border-top: 0;
    border-left: 2px dashed #6c757d;
    height: 24px;
    margin: 0.5rem 0;
  }
}

.border-box {
  border: 1px solid #e2e2e2;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 0 5px #1213;
}

.formSetp1 {
  display: grid;
  grid-template-columns: 1fr;
  width: 95%;
  margin: 24px auto;
}

.formSetp2 {
  border: 1px solid #e2e2e2;
  border-radius: 8px;
  padding: 16px;
  box-shadow: 0 0 5px #1213;
  width: 95%;
  margin: 24px auto;
}

.metaBox {
  width: 95%;
  margin: 24px auto;
}

.g-3 {
  gap: 16px;
}

.step-buttons {
  width: max(50%, 380px);
  margin: auto;
}

.step-divider {
  border: 2px dashed gray !important;
  width: 90px;
}
</style>
