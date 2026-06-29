<template>
  <div class="container mt-4">
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-list-nested"></i>
          <span>دسته بندی محصولات</span>
        </h3>
        <router-link to="/products/categories/create" class="btn btn-success">
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
                <th>دسته والد</th>
                <th>نمایش در صفحه اصلی</th>
                <th>نمایش لیست در صفحه اصلی</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="cat in categories" :key="cat.id">
                <td>{{ cat.id }}</td>
                <td>{{ cat.title }}</td>
                <td>{{ cat.parent?.title || '---' }}</td>
                <td>{{ cat.show_in_home ? "بله" : "خیر" }}</td>
                <td>{{ cat.show_products_in_home ? "بله" : "خیر" }}</td>
                <td>
                  <router-link :to="`/products/categories/${cat.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" @click="deleteCategory(cat.id)">
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
const categories = ref();
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/categories";

const getCategories = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    categories.value = flattenCategories(data.data);

  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// تابع برای مسطح کردن منوهای سلسله‌مراتبی
const flattenCategories = (categoryItems, level = 0, parent = null) => {
  let result = [];
  categoryItems.forEach((category) => {
    // اضافه کردن منوی فعلی به لیست
    result.push({
      ...category,
      level, // برای نمایش فاصله‌گذاری (indentation)
      parent, // برای دسترسی به والد
    });
    // اگر فرزندان وجود دارند، آن‌ها را هم به‌صورت بازگشتی اضافه کن
    if (category.all_children && category.all_children.length > 0) {
      result = result.concat(
        flattenCategories(category.all_children, level + 1, category)
      );
    }
  });
  return result;
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
        await axios.delete(`/categories/${id}`);
        Swal.fire("موفق", "دسته‌بندی حذف شد", "success");
        getCategories();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getCategories();
});
</script>