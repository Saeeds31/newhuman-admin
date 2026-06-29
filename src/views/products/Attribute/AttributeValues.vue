<template>
  <div class="container mt-4" v-if="checkPermission(['attributes_view'])">
    <div class="row">
      <!-- جدول -->
      <div class="col-md-7">
        <div class="card">
          <div class="card-header">
            <h3>
              لیست مقادیر ویژگی
              <b class="text-danger">
                {{ attribute }}

              </b>
            </h3>
          </div>
          <div class="card-body">
            <div v-if="loading" class="text-center py-4">
              <div class="spinner-border text-primary"></div>
            </div>
            <div v-else>
              <table class="table table-bordered">
                <thead>
                  <tr>
                    <th>شناسه</th>
                    <th>مقدار</th>
                    <th v-if="attributeType">
                      {{ attributeType == 'code' ? 'رنگ' : 'تصویر' }}
                    </th>
                    <th>عملیات</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="val in values" :key="val.id">
                    <td>{{ val.id }}</td>
                    <td>{{ val.value }}</td>
                    <td v-if="attributeType">
                      <div v-if="attributeType == 'code'" class="colorcode"
                        :style="{ backgroundColor: val.extra_value }">
                      </div>
                      <img width="64" :src="finderImage(val.extra_value)" v-else-if="val.extra_value"
                        class="imageExtra" />
                    </td>
                    <td>
                      <button class="btn btn-sm btn-warning me-2" @click="editValue(val)">
                        <i class="bi bi-pen"></i>
                        <span> ویرایش</span>
                      </button>
                      <button class="btn btn-sm btn-danger" @click="deleteValue(val.id)">
                        <i class="bi bi-trash3-fill"></i>
                        <span>حذف</span>
                      </button>
                    </td>
                  </tr>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>

      <!-- فرم -->
      <div class="col-md-5">
        <div class="card">
          <div class="card-header">
            {{ form.id ? "ویرایش مقدار" : "افزودن مقدار جدید" }}
          </div>
          <div class="card-body">
            <form @submit.prevent="saveValue">
              <div class="mb-3">
                <label class="form-label">مقدار</label>
                <input v-model="form.value" type="text" class="form-control" placeholder="مقدار ویژگی را وارد کنید" />
              </div>
              <div class="mb-3" v-if="attributeType">
                <label class="form-label">
                  {{ attributeType == 'code' ? "رنگ" : form.id ? "آپلود تصویر جدید" : "انتخاب تصویر" }} </label>
                <input v-if="attributeType == 'code'" v-model="form.extra_value" type="color" class="form-control"
                  placeholder="مقدار ویژگی را وارد کنید" />
                <VueFileAgent v-else-if="attributeType == 'image'" @select="imageLoaded" :maxFiles="1" accept=".pdf,.jpg,.png,.webp" theme="grid" deletable
                  sortable />
              </div>


              <button :disabled="loading" type="submit" class="btn btn-primary">
                <i class="bi bi-save2"></i>
                {{ form.id ? "به‌روزرسانی" : "ثبت" }}
              </button>
              <button v-if="form.id" type="button" class="btn btn-secondary ms-2" @click="resetForm">
                انصراف
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, reactive } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useRoute } from "vue-router";
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute();
const attributeId = route.params.id; // آیدی ویژگی از URL
const errors = reactive({})

const values = ref([]);
let loading = ref(false);

const form = ref({
  id: null,
  value: "",
  extra_value: ""
});
function finderImage(path) {
  return `${baseImageAddress}${path}`
}
function imageLoaded(files) {
  form.value.extra_value = files[0].file
}
let attribute = ref("");
let attributeType = ref("");
let currentUrl = `/attributes/${attributeId}/values`;
const getAttribute = async () => {
  const res = await axios.get(`/attributes/${attributeId}`)
  attribute.value = res.data.data.name
  attributeType.value = res.data.data.value_type
}
getAttribute();
const getValues = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
    values.value = data.data;
    currentUrl = url;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};


const saveValue = async () => {
  let formData = new FormData();
  formData.append("attribute_id", attributeId)
  formData.append("value", form.value.value)
  if (attributeType.value) {
    formData.append("extra_value", form.value.extra_value)
  }
  loading.value = true;
  try {
    if (form.value.id) {
      formData.append("_method", "PUT")
      await axios.post(`/attributes/${attributeId}/values/${form.value.id}`, formData);
      toast.success('مقدار ویرایش شد ✅')
      resetForm();
      getValues();
    } else {
      await axios.post(`/attributes/${attributeId}/values`, formData);
      toast.success('مقدار با موفقیت اضافه شد ✅')
      resetForm();
      getValues();
    }

  } catch (err) {
    if (err.response?.status === 422) {
      Object.assign(errors, err.response.data.errors)
    }
    toast.error(errors.value[0])
  } finally {
    loading.value = false;
  }
};

const editValue = (val) => {
  console.log(val);

  form.value = { ...val };
};

const resetForm = () => {
  form.value = { id: null, value: "" };
};

const deleteValue = (id) => {
  Swal.fire({
    title: "حذف مقدار",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/attributes/${attributeId}/values/${id}`);
        Swal.fire("موفق", "مقدار حذف شد", "success");
        getValues();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getValues();
});
</script>
<style>
.colorcode {
  width: 68px;
  height: 34px;
  border-radius: 8px;
}
</style>