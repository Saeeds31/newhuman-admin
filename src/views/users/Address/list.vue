<template>
    <div class="container addresses-page" v-if="checkPermission(['address_view'])">
        <!-- Header -->
        <div class="card mb-3">
            <div class="card-header d-flex justify-content-between align-items-center">
                <h3>
                    <i class="bi bi-house-add"></i>
                    <span>مدیریت آدرس‌ها</span>
                </h3>
                <router-link :to="`/users/${route.params.id}/addresses/create`" class="btn btn-primary">
                    <i class="bi bi-plus"></i>
                    <span>
                        افزودن آدرس
                    </span>
                </router-link>
            </div>
        </div>


        <!-- کارت‌ها -->
        <div class="row g-3">
            <div v-if="loading" class="text-center my-5 col-12">
                <div class="spinner-border" role="status"></div>
                <p class="mt-2">در حال بارگذاری...</p>
            </div>

            <div v-else-if="addresses && addresses.length" v-for="address in addresses" :key="address.id"
                class="col-md-4">
                <div class="card address h-100">
                    <div class="card-body">
                        <h5 class="card-title">{{ address.receiver_name }}</h5>
                        <p class="card-text">
                            <strong>استان:</strong> {{ address.province?.name ?? '-' }}<br>
                            <strong>شهر:</strong> {{ address.city?.name ?? '-' }}<br>
                            <strong>کدپستی:</strong> {{ address.postal_code }}<br>
                            <strong>آدرس:</strong> {{ address.address_line }}<br>
                            <strong>تلفن:</strong> {{ address.phone }}
                        </p>
                    </div>
                    <div class="card-footer text-end">
                        <router-link :to="`/users/${route.params.id}/addresses/${address.id}/edit`"
                            class="btn btn-sm btn-info">
                          <i class="bi bi-pen"></i>
                          <span>  ویرایش</span>
                        </router-link>
                    </div>
                </div>
            </div>
            <p class="bg-warning-subtle p-4 text-center " v-else>
                <i class="bi bi-database-fill-exclamation"></i>
                <span>هیچ داده ای برای نمایش وجود ندارد</span>
            </p>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import { useRoute } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute();
const addresses = ref([]);
const loading = ref(false);

const getAddresses = async () => {
    loading.value = true;
    try {
        const response = await axios.get(`/users/${route.params.id}/addresses`);
        addresses.value = response.data;
    } finally {
        loading.value = false;
    }
};

onMounted(() => {
    getAddresses();
});
</script>

<style scoped>
.address {
    transition: transform 0.2s;
}

.address:hover {
    transform: translateY(-5px);
}
</style>