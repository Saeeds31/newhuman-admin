<template>
  <div class="container mt-4 specifications-page" v-if="checkPermission(['specifications_view'])">
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-table"></i>
          <span>مدیریت مشخصه
          </span>
        </h3>
        <router-link to="/products/specification/create" class="btn btn-primary">
          <i class="bi bi-plus"></i>
          <span> افزودن مشخصه
          </span>
        </router-link>
      </div>
    </div>


    <!-- Table -->
    <div class="card">
      <div class="card-body table-responsive">
        <table class="table table-bordered align-middle text-center">
          <thead>
            <tr>
              <th>#</th>
              <th>عنوان مشخصه</th>
              <th>تعداد مقادیر</th>
              <th>عملیات</th>
            </tr>
          </thead>
          <tbody>
            <tr v-if="loading">
              <td colspan="4" class="text-center">
                <div class="spinner-border" role="status"></div>
              </td>
            </tr>
            <tr v-else v-for="spec in specifications.data" :key="spec.id">
              <td>{{ spec.id }}</td>
              <td>{{ spec.title }}</td>
              <td>{{ spec.values.length }}</td>
              <td>
                <router-link :to="`/products/specification/${spec.id}/edit`" class="btn btn-sm btn-info me-1">
                  <i class="bi bi-pen"></i>
                  <span> ویرایش</span>
                </router-link>
                <button class="btn btn-sm btn-danger" @click="confirmDelete(spec.id)">
                  <i class="bi bi-trash3-fill"></i>
                  <span>حذف</span>
                </button>
              </td>
            </tr>
            <tr v-if="!loading && specifications.data.length === 0">
              <td colspan="4" class="text-center">هیچ مشخصه‌ای یافت نشد</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    <b-pagination v-model="currentPage" :total-rows="specifications.total" v-if="specifications.last_page != 1"
      :per-page="specifications.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;

const specifications = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });
const currentPage = ref(1);

const getSpecifications = async (page = 1) => {
  loading.value = true;
  try {
    const response = await axios.get("/specifications", {
      params: {
        page,
        search: filters.value.search,
      },
    });
    specifications.value = response.data.data;
    currentPage.value = page;
  } finally {
    loading.value = false;
  }
};

const changePage = (page) => {
  if (page) getSpecifications(page);
  else currentUrl = "/specifications"
};

const confirmDelete = (id) => {
  Swal.fire({
    title: "حذف مشخصه",
    text: "آیا از حذف این مشخصه مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      await axios.delete(`/specifications/${id}`);
      getSpecifications(currentPage.value);
      Swal.fire("حذف شد!", "مشخصه با موفقیت حذف گردید.", "success");
    }
  });
};

onMounted(() => {
  getSpecifications();
});
</script>
