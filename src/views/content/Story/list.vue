<template>
  <div class="container mt-4" v-if="checkPermission(['story_view'])">
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-list-columns-reverse"></i>
          <span>مدیریت استوری</span>
        </h3>
        <router-link to="/content/stories/create" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن استوری
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
                <th>تصویر</th>
                <th>وضعیت</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="story in stories" :key="story.id">
                <td>{{ story.id }}</td>
                <td>
                  <span :style="{ 'padding-right': `${story.level * 20}px` }">
                    {{ story.title }}
                  </span>
                </td>
                <td>
                  <img width="64" :src="findImage(story.cover)" alt="">
                </td>
                <td>
                  <span class="text-white  p-2 rounded "
                    :class="{ 'bg-success': story.status == 'published', 'bg-primary': story.status == 'draft', 'bg-warning': story.status == 'archived' }">
                    {{ getLabel(story.status) }}
                  </span>
                </td>

                <td>
                  <router-link :to="`/content/stories/${story.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" @click="deletestory(story.id)">
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
const stories = ref([]);
const flattenedstories = ref([]);
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/stories";
function findImage(path) {
  return baseImageAddress + path
}
function getLabel(status) {
  if (status == 'draft') {
    return "پیشنویس"
  } else if (status == 'published') {
    return "منتشر شده"
  } else {
    return "آرشیو شده"

  }
}

// دریافت استوریها
const getstories = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    stories.value = data.data;
  } catch (err) {
    console.error(err);
    Swal.fire("خطا", "مشکلی در دریافت استوریها پیش آمد", "error");
  } finally {
    loading.value = false;
  }
};

// حذف استوری
const deletestory = (id) => {
  Swal.fire({
    title: "حذف استوری",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/stories/${id}`);
        Swal.fire("موفق", "استوری حذف شد", "success");
        getstories(); // به‌روزرسانی لیست استوریها
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getstories();
});
</script>

<style scoped>
/* برای بهبود نمایش فاصله‌گذاری در استوریهای فرزند */
td span {
  display: inline-block;
}
</style>