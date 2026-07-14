<template>
  <div class="container mt-4">
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-list-nested"></i>
          <span>دسته بندی فایل ها</span>
        </h3>
        <router-link v-if="checkPermission(['filecategory_store'])" to="/files/categories/create"
          class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن دسته‌بندی
          </span>
        </router-link>
      </div>
    </div>

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
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="cat in categories.data" :key="cat.id">
                <td>{{ cat.id }}</td>
                <td>{{ cat.title }}</td>
                <td>
                  <router-link v-if="checkPermission(['filecategory_update'])"
                    :to="`/files/categories/${cat.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" v-if="checkPermission(['filecategory_delete'])"
                    @click="deleteCategory(cat.id)">
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
const categories = ref({});
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/file-categories";

const getCategories = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    categories.value = data.data

  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};




const deleteCategory = (id) => {
  Swal.fire({
    title: "حذف دسته‌بندی",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/file-categories/${id}`);
        Swal.fire("موفق", "دسته‌بندی حذف شد", "success");
        getCategories();
      } catch (err) {
        Swal.fire("خطا", err.response.data.message, "error");

      }
    }
  });
};

onMounted(() => {
  getCategories();
});
</script>