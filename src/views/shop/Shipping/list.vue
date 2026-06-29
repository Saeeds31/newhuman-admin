<template>
    <div class="container shipping-methods" v-if="checkPermission(['shipping_view'])">
        <!-- فیلتر جستجو -->
        <div class="card mb-3">
            <div class="card-body">
                <div class="row g-2 align-items-center">
                    <div class="col-md-4">
                        <input v-model="filters.search" @keyup.enter="fetchData" type="text" class="form-control"
                            placeholder="جستجو بر اساس نام" />
                    </div>
                    <div class="col-md-2">
                        <button @click="fetchData" class="btn btn-primary w-100">
                            جستجو
                        </button>
                    </div>
                </div>
            </div>
        </div>

        <!-- دکمه افزودن -->
        <div class="d-flex justify-content-end mb-3">
            <router-link to="/shop/shipping/create" class="btn btn-success">
                افزودن روش ارسال
            </router-link>
        </div>

        <!-- جدول -->
        <div class="card">
            <div class="card-body table-responsive">
                <div v-if="loading" class="text-center p-4">
                    <div class="spinner-border text-primary" role="status"></div>
                </div>

                <table v-else class="table table-bordered table-hover">
                    <thead>
                        <tr>
                            <th>نام</th>
                            <th>توضیحات</th>
                            <th>هزینه </th>
                            <th>وضعیت</th>
                            <th>عملیات</th>
                        </tr>
                    </thead>
                    <tbody>
                        <tr v-for="method in shippingMethods" :key="method.id">
                            <td>{{ method.title }}</td>
                            <td>{{ method.description }}</td>
                            <td>{{ method.cost }} تومان</td>
                            <td>
                                <span :class="method.status ? 'badge bg-success' : 'badge bg-danger'">
                                    {{ method.status ? 'فعال' : 'غیرفعال' }}
                                </span>
                            </td>
                            <td>
                                <router-link :to="`/shop/shipping/${method.id}/edit`"
                                    class="btn btn-sm btn-primary me-2">
                                    <i class="bi bi-pen"></i>
                                    <span>  ویرایش</span>
                                </router-link>
                                <button @click="deleteMethod(method.id)" class="btn btn-sm btn-danger">
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
</template>

<script setup>
import { ref, onMounted } from "vue";
import Swal from "sweetalert2";
import axios from "axios";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const shippingMethods = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });
const fetchData = async (url = "/shipping-methods") => {
    loading.value = true;
    try {
        const { data } = await axios.get(url, { params: filters.value });
        shippingMethods.value = data.data;
    } finally {
        loading.value = false;
    }
};
const deleteMethod = async (id) => {
    const result = await Swal.fire({
        title: "آیا مطمئن هستید؟",
        text: "این عملیات قابل بازگشت نیست!",
        icon: "warning",
        showCancelButton: true,
        confirmButtonText: "بله، حذف شود",
        cancelButtonText: "انصراف",
    });

    if (result.isConfirmed) {
        await axios.delete(`/shipping-methods/${id}`);
        Swal.fire("حذف شد!", "روش ارسال با موفقیت حذف شد.", "success");
        fetchData();
    }
};

onMounted(() => {
    fetchData();
});
</script>