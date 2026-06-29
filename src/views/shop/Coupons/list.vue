<template>
  <div class="container mt-4" v-if="checkPermission(['coupon_view'])">

    <!-- باکس فیلتر -->
    <div class="card mb-3">
      <div class="card-body">
        <form @submit.prevent="getCoupons">
          <div class="row g-2">
            <div class="col-md-4">
              <input v-model="filters.code" type="text" class="form-control" placeholder="جستجو بر اساس کد کوپن" />
            </div>
            <div class="col-md-2">
              <button class="btn btn-primary w-100" type="submit">جستجو</button>
            </div>
          </div>
        </form>
      </div>
    </div>

    <!-- دکمه افزودن -->
    <div class="mb-3 text-end">
      <router-link to="/shop/coupons/create" class="btn btn-success">
        افزودن کوپن
      </router-link>
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
                <th>کد</th>
                <th>نوع</th>
                <th>مقدار</th>
                <th>تاریخ شروع</th>
                <th>تاریخ پایان</th>
                <th>وضعیت</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="coupon in coupons.data" :key="coupon.id">
                <td>{{ coupon.id }}</td>
                <td>{{ coupon.code }}</td>
                <td>{{ coupon.type }}</td>
                <td>{{ coupon.value }}</td>
                <td>{{ formatDate(coupon.start_date) }}</td>
                <td>{{ formatDate(coupon.end_date) }}</td>
                <td>
                  <span :class="coupon.status ? 'badge bg-success' : 'badge bg-secondary'">
                    {{ coupon.status ? 'فعال' : 'غیرفعال' }}
                  </span>
                </td>
                <td>
                  <router-link :to="`/shop/coupons/${coupon.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span>  ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" @click="deleteCoupon(coupon.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>

          <!-- صفحه بندی -->
          <b-pagination v-model="currentPage" :total-rows="coupons.total" v-if="coupons.last_page != 1"
            :per-page="coupons.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>
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
const coupons = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ code: "" });
let currentUrl = "/coupons";

const getCoupons = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    coupons.value = data.data;
    currentUrl = url;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const changePage = (page) => {
  if (page) getCoupons(`${currentUrl}?page=${page}`);
  else currentUrl = "/copons"
};

const deleteCoupon = (id) => {
  Swal.fire({
    title: "حذف کوپن",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/coupons/${id}`);
        Swal.fire("موفق", "کوپن حذف شد", "success");
        getCoupons();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

const formatDate = (date) => {
  return new Date(date).toLocaleDateString("fa-IR");
};

onMounted(() => {
  getCoupons();
});
</script>