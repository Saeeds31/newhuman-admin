<template>
  <div class="container mt-4" v-if="checkPermission(['file_view'])">
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-files"></i>
          <span>مدیریت فایل</span>
        </h3>
        <router-link v-if="checkPermission(['file_store'])" to="/files/create" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن فایل
          </span>
        </router-link>
      </div>
    </div>
    <!-- جدول -->
    <div class="card">
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary"></div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>عنوان</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="file in files.data" :key="file.id">
                <td>{{ file.id }}</td>
                <td>{{ file.title }}</td>
                <td>
                  <router-link v-if="checkPermission(['file_update'])" :to="`/files/${file.id}/edit`"
                    class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" v-if="checkPermission(['file_delete'])"
                    @click="deletefile(file.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                  </button>
                  <button class="btn btn-sm btn-primary me-2" @click="copyFile(file.file)">
                    <i class="bi bi-copy"></i>
                    <span>کپی</span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>


          <b-pagination v-model="currentPage" :total-rows="files.total" v-if="files.last_page != 1"
            :per-page="files.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useRoute, useRouter, } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
import { toast } from 'vue3-toastify'
import 'vue3-toastify/dist/index.css'
const store = useAdmin();
const checkPermission = store.checkPermission;
const currentPage = ref(1)
const router = useRouter();
const route = useRoute();
const files = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ search: "", status: "" });
let currentUrl = "/files";

async function getFiles(url = currentUrl) {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    files.value = data.data;
    currentPage.value = data.current_page
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};
async function copyFile(path) {
  let text = window.baseImageAddress + path;
  try {
    await navigator.clipboard.writeText(text)
    toast.success('آدرس فایل کپی شد ✅')
  } catch (err) {
    let text = window.baseImageAddress + path;
    if (text) {
      const textarea = document.createElement('textarea')
      textarea.value = sheba
      document.body.appendChild(textarea)
      textarea.select()
      document.execCommand('copy')
      document.body.removeChild(textarea)
      toast.success('آدرس فایل کپی شد ✅')
    } else {
      console.log('خطا در کپی کردن')
    }
  }
}


function changePage(selectedPage) {
  if (selectedPage) {
    router.replace({ name: route.name, query: { page: selectedPage } })
    getFiles(`${currentUrl}?page=${selectedPage}`)
  }
  else currentUrl = "/portfolios"
}
const deletefile = (id) => {
  Swal.fire({
    title: "حذف نمونه کار",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/files/${id}`);
        Swal.fire("موفق", "نمونه کار حذف شد", "success");
        getFiles();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  currentPage.value = route.query.page ?? 1;
  getFiles(`${currentUrl}?page=${currentPage.value}`);
});
</script>