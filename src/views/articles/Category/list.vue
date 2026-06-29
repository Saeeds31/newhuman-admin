<template>
    <div class="container mt-4" v-if="checkPermission(['articlecategory_view'])">
        <div class="card mb-2">
            <div class="card-header d-flex justify-content-between align-items-center mb-3">
                <h3>
                    <i class="bi bi-list-stars"></i>
                    <span>مدیریت دسته بندی</span>
                </h3>
                <router-link to="/articles/categories/create" class="btn btn-primary">

                    <i class="bi bi-plus"></i>
                    <span> افزودن دسته بندی</span>
                </router-link>
            </div>
        </div>

        <b-table class="table table-bordered table-striped" striped hover :items="categories.data" :fields="fields">
            <template #cell(parent)="data">
                {{ data.item.parent ? data.item.parent.title : '-' }}
            </template>

            <template #cell(actions)="data">
                <router-link :to="`/articles/categories/${data.item.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span> </router-link>
                <button class="btn btn-sm btn-danger" @click="confirmDelete(data.item.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                </button>
            </template>
        </b-table>

        <b-pagination v-model="currentPage" :total-rows="categories.total" v-if="categories.last_page != 1"
            :per-page="categories.per_page" @Update:modelValue="fetchCategories" align="center"
            class="mt-3"></b-pagination>
    </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import Swal from 'sweetalert2'
import { useRoute, useRouter } from 'vue-router'
let router = useRouter();
let route = useRoute();
import { useAdmin } from '@/stores/modules/admin';
const store = useAdmin();
const checkPermission = store.checkPermission;
const categories = ref({
    data: [],
    total: 0,
    per_page: 10,
    current_page: 1,
})
const currentPage = ref(1)

const fields = [
    { key: 'id', label: 'شناسه' },
    { key: 'title', label: 'عنوان' },
    { key: 'slug', label: 'اسلاگ' },
    { key: 'parent', label: 'والد' },
    { key: 'actions', label: 'عملیات' },
]

const fetchCategories = async (page = 1) => {
    try {
        router.replace({ name: route.name, query: { page: page } })
        const res = await axios.get(`/article-categories?page=${page}`)
        categories.value = res.data.data
        currentPage.value = res.data.data.current_page
    } catch (error) {
        console.error(error)
    }
}

const confirmDelete = (id) => {
    Swal.fire({
        title: 'آیا مطمئن هستید?',
        text: "این عملیات بازگشت پذیر نیست!",
        icon: 'warning',
        showCancelButton: true,
        confirmButtonColor: '#d33',
        cancelButtonColor: '#3085d6',
        confirmButtonText: 'بله انجام شود!',
        cancelButtonText: 'لغو',
    }).then((result) => {
        if (result.isConfirmed) {
            deleteCategory(id)
        }
    })
}

const deleteCategory = async (id) => {
    try {
        await axios.delete(`/article-categories/${id}`)
        Swal.fire('پاک شد!', 'با موفقیت حذف شد.', 'success')
        fetchCategories(currentPage.value)
    } catch (error) {
        console.error(error)
        Swal.fire('Error!', error.response.data.message ?? 'خطایی در حذف رخ داد', 'error')
    }
}

onMounted(() => {
    fetchCategories()
})


</script>