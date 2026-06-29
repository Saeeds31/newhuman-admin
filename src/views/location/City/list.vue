<template>
  <div class="container mt-4" v-if="checkPermission(['province_view'])">
    <!-- باکس فیلتر -->
    <div class="card mb-3">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-pin-map"></i>
          <span>مدیریت شهرها</span>
        </h3>
        <router-link to="/location/cities/create" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span class="mx-2">
            افزودن شهر
          </span>
        </router-link>
      </div>
      <div class="card-body">
        <form @submit.prevent="getCities()">
          <div class="row g-2">
            <div class="col-md-4">
              <input v-model="filters.search" type="text" class="form-control" placeholder="جستجو بر اساس نام شهر" />
            </div>
            <div class="col-md-4">
              <select v-model="filters.province_id" class="form-select">
                <option value="">همه استان‌ها</option>
                <option v-for="prov in provinces" :key="prov.id" :value="prov.id">
                  {{ prov.name }}
                </option>
              </select>
            </div>
            <div class="col-md-2">
              <button class="btn btn-primary w-100" type="submit">
                <i class="bi bi-search"></i>
                <span class="mx-2">
                  جستجو
                </span>
              </button>
            </div>
          </div>
        </form>
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
                <th>نام شهر</th>
                <th>استان</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="city in cities.data" :key="city.id">
                <td>{{ city.id }}</td>
                <td>{{ city.name }}</td>
                <td>{{ city.province?.name || '---' }}</td>
                <td>
                  <router-link :to="`/location/cities/${city.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" @click="deleteCity(city.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>

          <!-- صفحه بندی -->
          <b-pagination v-model="currentPage" :total-rows="cities.total" v-if="cities.last_page != 1"
            :per-page="cities.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

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
const cities = ref({ data: [], meta: null });
const provinces = ref([]);
const loading = ref(false);
const filters = ref({ search: "", province_id: "" });
let currentUrl = "/cities";

// دریافت لیست شهرها
const getCities = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    cities.value = data.data;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

// دریافت لیست استان‌ها برای فیلتر
const getProvinces = async () => {
  try {
    const { data } = await axios.get("/provinces");
    provinces.value = data.data;
  } catch (err) {
    console.error(err);
  }
};

const changePage = (page) => {
  if (page) getCities(`${currentUrl}?page=${page}`);
  else currentUrl = "/cities"
};
const deleteCity = (id) => {
  Swal.fire({
    title: "حذف شهر",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/cities/${id}`);
        Swal.fire("موفق", "شهر حذف شد", "success");
        getCities();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getProvinces();
  getCities();
});
</script>