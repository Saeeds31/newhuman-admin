<template>
  <div class="product-create container py-4" v-if="checkPermission(['file_store'])">
    <div class="bg-gray">
      <h3 class=" p-2">
        <i class="bi bi-info"></i>
        <span>ثبت اطلاعات فایل</span>
      </h3>
      <form @submit.prevent="submitData">
        <div class="formSetp1  g-3">
          <div class=" border-box row">
            <div class="col-md-6 mb-3">
              <label class="form-label">عنوان فایل</label>
              <input v-model="form.title" type="text" class="form-control" />
              <span v-if="errors.title" class="text-danger">{{ errors.title[0] }}</span>
            </div>


            <div class="col-md-6 mb-3">
              <label class="form-label">دسته‌بندی‌</label>
              <Treeselect v-model="form.category_id" :multiple="false" :options="categoryOptions"
                :normalizer="normalizer" />
              <span v-if="errors.categories" class="text-danger">{{ errors.categories[0] }}</span>
            </div>

            <div class="col-md-12 mb-3">
              <label class="form-label">فایل </label>
              <VueFileAgent @select="imageLoaded1" :maxFiles="1" theme="grid" deletable sortable />
              <span v-if="errors.file" class="text-danger">{{ errors.file[0] }}</span>
            </div>
          </div>
        </div>
        <button :disabled="loading" type="submit" class="btn btn-primary mt-3">
          <i class="bi bi-save2"></i>
          <span class="mx-2">ذخیره فایل</span>
        </button>
      </form>
    </div>



  </div>
</template>

<script setup>
import { ref, watch, onMounted, reactive, nextTick } from 'vue'
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import axios from 'axios'
import Editor from '@/components/shared/editor.vue';
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const portfolio = ref(null)

function imageLoaded1(files) {
  form.value.file = files[0].file
}
let loading = ref(false);

const form = ref({
  title: '', category_id: '', file: "",
})
const errors = ref({})
const categoryOptions = ref([])
const normalizer = node => ({ id: node.id, label: node.title, children: node.children })

onMounted(() => {
  loadCategories();
})


async function loadCategories() {
  const res = await axios.get('/file-categories')
  categoryOptions.value = res.data.data.data
}

async function submitData() {
  errors.value = {}
  loading.value = true;
  try {
    const formData = new FormData()
    Object.keys(form.value).forEach(key => {
      formData.append(key, form.value[key])
    })
    const res = await axios.post('/files', formData)
    portfolio.value = res.data
    toast.success('فایل با موفقیت ذخیره شد!')
  } catch (e) {
    console.log(e);
    if (e.response?.data?.errors) errors.value.step1 = e.response.data.errors
    toast.error('خطا در ذخیره نمونه کار')
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
    border-left: 2px dashed #0d6efd;
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

.speci,
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
  border: 2px dashed #0d6efd !important;
  width: 90px;
}
</style>
