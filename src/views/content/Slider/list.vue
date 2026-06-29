<template>
  <div class="container mt-4 sliders" v-if="checkPermission(['slider_view'])">
    <div class="card mb-3">
      <div class="card-header d-flex align-items-center justify-content-between mb-3">
        <h3>
          <i class="bi bi-sliders"></i>
          <span>مدیریت اسلایدرها</span>
        </h3>
        <router-link to="/content/sliders/create" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن اسلایدر
          </span>
        </router-link>
      </div>
    </div>

    <!-- افزودن -->


    <!-- جدول -->
    <div class="card">
      <div class="card-body table-responsive">
        <div v-if="loading" class="text-center p-4">
          <div class="spinner-border text-primary"></div>
        </div>

        <table v-else class="table table-bordered table-hover">
          <thead>
            <tr>
              <th>عنوان</th>
              <th>نوع</th>
              <th>تصویر</th>
              <th>عملیات</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="slider in sliders" :key="slider.id">
              <td>{{ slider.title }}</td>
              <td>{{ slider.type == "desktop" ? "دسکتاپ" : "موبایل" }}</td>
              <td>
                <img v-if="slider.image" :src="slider.image" alt="" class="img-thumbnail" style="max-width:100px;">
              </td>
              <td>
                <router-link :to="`/content/sliders/${slider.id}/edit`" class="btn btn-sm btn-primary me-2"> <i
                    class="bi bi-pen"></i>
                  <span> ویرایش</span></router-link>
                <button @click="deleteSlider(slider.id)" class="btn btn-sm btn-danger"> <i
                    class="bi bi-trash3-fill"></i>
                  <span>حذف</span></button>
              </td>
            </tr>
          </tbody>
        </table>

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
const sliders = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });
let currentUrl = "/sliders";

const fetchData = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    sliders.value = data.data.map(slide => ({ id: slide.id, title: slide.title, type: slide.type, image: baseImageAddress + slide.image, }));

  } finally {
    loading.value = false;
  }
};


const deleteSlider = async (id) => {
  const result = await Swal.fire({
    title: "آیا مطمئن هستید؟",
    text: "این عملیات قابل بازگشت نیست!",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  });

  if (result.isConfirmed) {
    await axios.delete(`/sliders/${id}`);
    Swal.fire("حذف شد!", "اسلایدر با موفقیت حذف شد.", "success");
    fetchData();
  }
};

onMounted(() => {
  fetchData();
});
</script>