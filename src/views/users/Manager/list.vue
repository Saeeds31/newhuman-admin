<template>
    <div class="users-page container mt-4 " v-if="checkPermission(['manager_view'])">
        <div class="card mb-3">
            <div class="card-header d-flex justify-content-between align-items-center mb-3">
                <h3>
                    <i class="bi bi-person-workspace"></i>
                    <span>
                        مدیریت مدیران
                    </span>
                </h3>
                <router-link to="/users/managers/create" class="btn btn-primary">
                    <i class="bi bi-plus"></i>
                    <span>
                        افزودن مدیر
                    </span>
                </router-link>
            </div>
            <div class="card-body">
                <input v-model="filters.search" @input="getManagers" type="text" class="form-control"
                    placeholder="جستجو بر اساس نام یا موبایل" />
            </div>
        </div>

        <!-- Table -->
        <div class="card">
            <div class="card-body">
                <div v-if="loading" class="text-center my-5">
                    <div class="spinner-border" role="status"></div>
                    <p class="mt-2">در حال بارگذاری...</p>
                </div>

                <div v-else>
                    <table class="table table-bordered table-striped">
                        <thead>
                            <tr>
                                <th>شناسه</th>
                                <th>نام کامل</th>
                                <th>موبایل</th>
                                <th>عملیات</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="user in users" :key="user.id">
                                <td>{{ user.id }}</td>
                                <td>{{ user.full_name }}</td>
                                <td>{{ user.mobile }}</td>
                                <td>
                                    <router-link :to="`/users/managers/${user.id}/edit`" class="btn btn-sm btn-info">
                                        <i class="bi bi-pen"></i>
                                        <span> ویرایش</span>
                                    </router-link>
                                    <button class="btn btn-sm btn-danger ms-2" @click="confirmDelete(user.id)">
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
import { useRoute, useRouter } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const router = useRouter();
const route = useRoute();
const users = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });
let abortController = null;

const getManagers = async () => {
    loading.value = true;
    // اگر درخواست قبلی وجود داشت، کنسل کن
    if (abortController) {
        abortController.abort();
    }

    abortController = new AbortController();
    try {
        const response = await axios.get("/user-managers", {
            params: {
                search: filters.value.search,
            },
            signal: abortController.signal,

        });
        users.value = response.data;
    } finally {
        loading.value = false;
    }
};
onMounted(() => {
    getManagers();
});
</script>