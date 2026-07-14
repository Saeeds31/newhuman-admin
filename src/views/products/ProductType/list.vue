<template>
  <div class="container mt-4" v-if="checkPermission(['producttype_view'])">
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center">
        <h3>
          <i class="bi bi-tags"></i>
          <span>مدیریت دسته‌بندی‌ها</span>
        </h3>
        <router-link v-if="checkPermission(['producttype_store'])" to="/products/product-types/create"
          class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>افزودن دسته‌بندی</span>
        </router-link>
      </div>
    </div>

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
                <th>نام</th>
                <th>slug</th>
                <th>وضعیت</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in items.data" :key="item.id">
                <td>{{ item.id }}</td>
                <td>{{ item.name }}</td>
                <td>{{ item.slug }}</td>
                <td>
                  <span :class="item.is_active ? 'badge bg-success' : 'badge bg-danger'">
                    {{ item.is_active ? 'فعال' : 'غیرفعال' }}
                  </span>
                </td>
                <td>
                  <router-link v-if="checkPermission(['producttype_update'])"
                    :to="`/products/product-types/${item.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" v-if="checkPermission(['producttype_delete'])"
                    @click="deleteItem(item.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>

          <b-pagination v-model="currentPage" :total-rows="items.total" v-if="items.last_page != 1"
            :per-page="items.per_page" @Update:modelValue="changePage" align="center" class="mt-3">
          </b-pagination>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { useRoute, useRouter } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';

const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute();
const router = useRouter();
const currentPage = ref(1);
const items = ref({ data: [], total: 0, last_page: 1, per_page: 20 });
const loading = ref(false);

async function getItems(url = "/product-types") {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
    items.value = data.data;
    currentPage.value = data.data.current_page;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
}

function changePage(selectedPage) {
  router.replace({ name: route.name, query: { page: selectedPage } });
  getItems(`/product-types?page=${selectedPage}`);
}

const deleteItem = (id) => {
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
        await axios.delete(`/product-types/${id}`);
        Swal.fire("موفق", "دسته‌بندی حذف شد", "success");
        getItems();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  currentPage.value = route.query.page ?? 1;
  getItems();
});
</script>