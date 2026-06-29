<template>
    <div class="container mt-3 wallets-page" v-if="checkPermission(['wallet_view'])">


        <!-- فیلتر -->
        <div class="card mb-3">
            <div class="card-header d-flex justify-content-between align-items-center mb-3">
                <h3>
                    <i class="bi bi-wallet"></i>
                    <span>مدیریت کیف پول‌ها</span>
                </h3>
            </div>
            <div class="card-body">
                <input v-model="filters.search" @input="getWallets" type="text" class="form-control"
                    placeholder="جستجو بر اساس نام کاربر  " />
            </div>
        </div>

        <!-- جدول -->
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
                                <th>کاربر</th>
                                <th>موجودی (تومان)</th>
                                <th>عملیات</th>
                            </tr>
                        </thead>
                        <tbody>
                            <tr v-for="wallet in wallets.data" :key="wallet.id">
                                <td>{{ wallet.id }}</td>
                                <td>{{ wallet.user?.full_name ?? '-' }}</td>
                                <td>{{ Number(wallet.balance).toLocaleString('fa-ir') }}</td>
                                <td>
                                    <button class="btn btn-sm btn-success me-2" @click="openModal(wallet, 'deposit')">
                                        <i class="bi bi-plus"></i>
                                        افزایش موجودی
                                    </button>
                                    <button class="btn btn-sm btn-warning" @click="openModal(wallet, 'withdraw')">
                                        <i class="bi bi-dash"></i>
                                        کاهش موجودی

                                    </button>
                                </td>
                            </tr>
                        </tbody>
                    </table>
                    <b-pagination v-model="currentPage" :total-rows="wallets.total" v-if="wallets.last_page != 1"
                        :per-page="wallets.per_page" @Update:modelValue="changePage" align="center"
                        class="mt-3"></b-pagination>


                </div>
            </div>
        </div>

        <!-- Modal -->
        <div class="modal fade" id="walletModal" tabindex="-1" aria-hidden="true" ref="walletModal">
            <div class="modal-dialog">
                <div class="modal-content">
                    <div class="modal-header">
                        <h5 class="modal-title">{{ modalTitle }}</h5>
                        <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="بستن"></button>
                    </div>
                    <div class="modal-body">
                        <div class="mb-3">
                            <label class="form-label">مبلغ

                                <span v-if="form.amount">
                                    {{ Number(form.amount).toLocaleString('fa-ir') }}
                                </span>
                                (تومان)</label>
                            <input v-model="form.amount" type="number" class="form-control"
                                placeholder="مثلاً 100000" />
                        </div>
                        <div class="mb-3">
                            <label class="form-label">توضیح</label>
                            <textarea v-model="form.description" class="form-control" rows="3"></textarea>
                        </div>
                    </div>
                    <div class="modal-footer">
                        <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">
                            <i class="bi bi-x"></i>
                            انصراف
                        </button>
                        <button type="button" class="btn btn-primary" @click="submitForm">
                            <i class="bi bi-save2"></i>
                            ثبت
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import { Modal } from "bootstrap";
import { useRoute, useRouter } from "vue-router";
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
let router = useRouter();
let route = useRoute();
const wallets = ref({ data: [] });
const loading = ref(false);
const filters = ref({ search: "" });
const currentPage = ref(1);

const modalInstance = ref(null);
const modalTitle = ref("");
const actionType = ref(""); // deposit | withdraw
const selectedWallet = ref(null);
const form = ref({
    amount: "",
    description: "",
});
let abortController = null;

const getWallets = async (page = 1) => {
    loading.value = true;
    if (abortController) {
        abortController.abort();
    }

    abortController = new AbortController();
    try {
        const response = await axios.get("/wallets", {
            params: {
                page,
                search: filters.value.search,
            },
            signal: abortController.signal,
        });
        wallets.value = response.data;
        currentPage.value = page;
    } finally {
        loading.value = false;
    }
};


const changePage = (page) => {
    if (page) {
        router.replace({ name: route.name, query: { page: page } })
        getWallets(page)
    }
    else currentUrl = "/products"
};
const openModal = (wallet, type) => {
    selectedWallet.value = wallet;
    actionType.value = type;
    modalTitle.value = type === "deposit" ? "افزایش موجودی" : "کاهش موجودی";
    form.value.amount = "";
    form.value.description = "";
    modalInstance.value = new Modal(document.getElementById("walletModal"));
    modalInstance.value.show();
};

const submitForm = async () => {
    if (!form.value.amount) {
        Swal.fire("خطا", "لطفاً مبلغ را وارد کنید", "error");
        return;
    }
    let formData = new FormData();
    formData.append("wallet_id", selectedWallet.value.id)
    formData.append("amount", form.value.amount)
    formData.append("description", form.value.description)
    formData.append("type", actionType.value == "deposit" ? 'credit' : 'debit')
    try {
        await axios.post(`/wallets/${selectedWallet.value.id}/transactions`, formData);
        modalInstance.value.hide();
        getWallets(currentPage.value);
        Swal.fire("موفق", "عملیات با موفقیت انجام شد", "success");
    } catch (error) {
        Swal.fire("خطا", "مشکلی رخ داد", "error");
    }
};

onMounted(() => {
    getWallets();
});
</script>