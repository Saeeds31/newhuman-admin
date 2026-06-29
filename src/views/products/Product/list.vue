<template>
  <div class="container mt-4" v-if="checkPermission(['product_view'])">
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-box-fill"></i>
          <span>مدیریت محصولات</span>
        </h3>
        <router-link to="/products/create" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن محصول
          </span>
        </router-link>
      </div>
      <div class="card-body">
        <form @submit.prevent="getProducts()">
          <div class="row g-2">
            <div class="col-md-4">
              <input v-model="filters.search" type="text" class="form-control" placeholder="جستجو بر اساس نام محصول" />
            </div>
            <div class="col-md-2">
              <select v-model="filters.status" class="form-select">
                <option value="">همه وضعیت‌ها</option>
                <option value="published">فعال</option>
                <option value="unpublished">غیرفعال</option>
              </select>
            </div>
            <div class="col-md-2">
              <button class="btn btn-primary w-100" type="submit">جستجو</button>
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
                <th>عنوان</th>
                <th>قیمت</th>
                <th>موجودی</th>
                <th>تخفیف</th>
                <th>وضعیت</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="product in products.data" :key="product.id">
                <td>{{ product.id }}</td>
                <td>{{ product.title }}</td>
                <td>{{ product.price }}</td>
                <td>{{ product.stock }}</td>
                <td>
                  <span v-if="product.discount_value">
                    {{ product.discount_value }}
                    {{ product.discount_type === 'percent' ? '%' : 'تومان' }}
                  </span>
                  <span v-else>—</span>
                </td>
                <td>
                  <span :class="product.status ? 'badge bg-success' : 'badge bg-secondary'">
                    {{ product.status ? 'فعال' : 'غیرفعال' }}
                  </span>
                </td>
                <td>
                  <router-link :to="`/products/${product.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" @click="deleteProduct(product.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>


          <b-pagination v-model="currentPage" :total-rows="products.total" v-if="products.last_page != 1"
            :per-page="products.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

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
const store = useAdmin();
const checkPermission = store.checkPermission;
const currentPage = ref(1)
const router = useRouter();
const route = useRoute();
const products = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ search: "", status: "" });
let currentUrl = "/products";

async function getProducts(url = currentUrl) {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    products.value = data;
    currentPage.value = data.current_page
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

function changePage(selectedPage) {
  if (selectedPage) {
    router.replace({ name: route.name, query: { page: selectedPage } })
    getProducts(`${currentUrl}?page=${selectedPage}`)
  }
  else currentUrl = "/products"
}
const deleteProduct = (id) => {
  Swal.fire({
    title: "حذف محصول",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/products/${id}`);
        Swal.fire("موفق", "محصول حذف شد", "success");
        getProducts();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  currentPage.value = route.query.page ?? 1;
  getProducts(`${currentUrl}?page=${currentPage.value}`);
});
</script>