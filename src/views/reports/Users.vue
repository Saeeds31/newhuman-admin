<template>
    <b-container fluid class="py-4" v-if="checkPermission(['report_users'])">
        <!-- عنوان و لودر -->
        <div class="card mb-3">
            <div class="card-header d-flex justify-content-between align-items-center mb-3">
                <h3>
                    <i class="bi bi-reception-4"></i>
                    <span>
                        گزارش کاربران
                    </span>
                </h3>
                <b-spinner small v-if="loading"></b-spinner>
            </div>
            <div class="card-body">
                <b-form @submit.prevent="fetchData()">
                    <b-row>

                        <b-col md="3">
                            <b-form-group label="شماره موبایل">
                                <b-form-input v-model="filters.mobile" placeholder="مثلاً 0912..."></b-form-input>
                            </b-form-group>
                        </b-col>
                        <b-col md="3">
                            <b-form-group label="کد ملی">
                                <b-form-input v-model="filters.national_code" placeholder="کد ملی"></b-form-input>
                            </b-form-group>
                        </b-col>
                        <b-col md="3">
                            <b-form-group label="از تاریخ ثبت‌نام">
                                <date-picker display-format="jYYYY/jMM/jDD" format="YYYY-MM-DD"
                                    v-model="filters.date_from"></date-picker>

                            </b-form-group>
                        </b-col>
                        <b-col md="3">
                            <b-form-group label="تا تاریخ ثبت‌نام">
                                <date-picker display-format="jYYYY/jMM/jDD" format="YYYY-MM-DD"
                                    v-model="filters.date_to"></date-picker>

                            </b-form-group>
                        </b-col>
                        <b-col md="3">
                            <b-form-group label="از تاریخ تولد">
                                <date-picker display-format="jYYYY/jMM/jDD" format="YYYY-MM-DD"
                                    v-model="filters.birth_date_from"></date-picker>

                            </b-form-group>
                        </b-col>
                        <b-col md="3">
                            <b-form-group label="تا تاریخ تولد">
                                <date-picker display-format="jYYYY/jMM/jDD" format="YYYY-MM-DD"
                                    v-model="filters.birth_date_to"></date-picker>

                            </b-form-group>
                        </b-col>
                    </b-row>
                    <b-button type="submit" variant="primary">
                        <i class="bi bi-save2"></i>
                        <span>

                            اعمال فیلتر
                        </span>

                    </b-button>
                </b-form>
            </div>
        </div>


        <!-- جدول -->
        <b-card>
            <b-table :items="users.data" :fields="fields" striped hover small responsive>
                <template #cell(roles)="data">
                    <span v-for="role in data.item.roles" :key="role.id" class="badge bg-info me-1">
                        {{ role.name }}
                    </span>
                </template>
                <template #cell(wallet)="data">
                    <span v-if="data.item.wallet" class="text-success">دارد</span>
                    <span v-else class="text-danger">ندارد</span>
                </template>
                <template #cell(addresses)="data">
                    {{ data.item.addresses.length }} آدرس
                </template>
            </b-table>

            <!-- pagination -->
            <b-pagination v-model="page" :total-rows="users.total" :per-page="users.per_page" align="center"
                class="mt-3" @change="fetchData" />
        </b-card>
    </b-container>
</template>

<script setup>
import { ref, reactive, onMounted } from "vue";
import axios from "axios";

import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const users = ref({ data: [] });
const loading = ref(false);
const page = ref(1);

const filters = reactive({
    role_id: "",
    mobile: "",
    national_code: "",
    date_from: "",
    date_to: "",
    birth_date_from: "",
    birth_date_to: "",
    has_wallet: "",
    has_address: "",
});

const fields = [
    { key: "id", label: "شناسه" },
    { key: "full_name", label: "نام کامل" },
    { key: "mobile", label: "موبایل" },
    { key: "roles", label: "نقش‌ها" },
    { key: "wallet", label: "کیف پول" },
    { key: "addresses", label: "آدرس‌ها" },
    { key: "created_at", label: "تاریخ ثبت‌نام" },
];

const walletOptions = [
    { value: "", text: "همه" },
    { value: true, text: "دارد" },
    { value: false, text: "ندارد" },
];

const addressOptions = [
    { value: "", text: "همه" },
    { value: true, text: "دارد" },
    { value: false, text: "ندارد" },
];

const fetchData = async () => {
    loading.value = true;
    try {
        const { data } = await axios.get("/reports/users", {
            params: { ...filters, page: page.value },
        });
        users.value = data;
    } finally {
        loading.value = false;
    }
};

onMounted(fetchData);
</script>