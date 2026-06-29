<template>
  <div class="container mt-4" v-if="checkPermission(['banner_view'])">

    <!-- باکس فیلتر -->
    <div class="card mb-3">
      <div class="card-header d-flex align-items-center justify-content-between mb-3 text-end">
        <h3>
          <i class="bi bi-images"></i>
          <span> مدیریت بنرها</span>
        </h3>
        <router-link to="/content/banners/create" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن بنر
          </span>
        </router-link>
      </div>
    </div>

    <!-- دکمه افزودن -->


    <!-- جدول -->
    <div class="card">
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">در حال بارگذاری...</span>
          </div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>عنوان</th>
                <th>موقعیت</th>
                <th>وضعیت</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="banner in banners" :key="banner.id">
                <td>{{ banner.id }}</td>
                <td>{{ banner.title }}</td>
                <td>{{ postitionTitle(banner.position) }}</td>
                <td>
                  <span :class="banner.status ? 'badge bg-success' : 'badge bg-secondary'">
                    {{ banner.status ? 'فعال' : 'غیرفعال' }}
                  </span>
                </td>
                <td>
                  <router-link :to="`/content/banners/${banner.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" @click="deleteBanner(banner.id)">
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
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const banners = ref([]);
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/banners";
let positions = [
  { id: 'after_discount', label: "بعد از تخفیف ها" },
  { id: 'after_new_product', label: "بعد از جدیدترین ها" },
  { id: 'before_blog', label: "قبل از بلاگ" },
];
function postitionTitle(pos) {
  let position = positions.find((p) => p.id == pos)
  return position ? position.label : "-"
}
const getBanners = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    banners.value = data.data;
    currentUrl = url;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};




const deleteBanner = (id) => {
  Swal.fire({
    title: "حذف بنر",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/banners/${id}`);
        Swal.fire("موفق", "بنر حذف شد", "success");
        getBanners();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getBanners();
});
</script>