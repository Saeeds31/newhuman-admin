<template>
  <div class="container mt-4" v-if="checkPermission(['attributes_view'])">

    <!-- دکمه افزودن ویژگی -->
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-list-check"></i>
          <span>لیست ویژگی ها</span>
        </h3>
        <router-link to="/products/attributes/create" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن ویژگی

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
                <th>نام ویژگی</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="attr in attributes" :key="attr.id">
                <td>{{ attr.id }}</td>
                <td>{{ attr.name }}</td>
                <td>
                  <router-link :to="`/products/attributes/${attr.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>

                  <router-link :to="`/products/attributes/${attr.id}/values`" class="btn btn-sm btn-info me-2">
                    <i class="bi bi-plus"></i>
                    <span>
                      افزودن مقدار
                    </span>
                  </router-link>

                  <button class="btn btn-sm btn-danger" @click="deleteAttribute(attr.id)">
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
const attributes = ref([]);
const loading = ref(false);
let currentUrl = "/attributes";

const getAttributes = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
    attributes.value = data.data;
    currentUrl = url;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};



const deleteAttribute = (id) => {
  Swal.fire({
    title: "حذف ویژگی",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/attributes/${id}`);
        Swal.fire("موفق", "ویژگی حذف شد", "success");
        getAttributes();
      } catch (err) {
        Swal.fire("خطا", err.response.data.message, "error");
      }
    }
  });
};

onMounted(() => {
  getAttributes();
});
</script>