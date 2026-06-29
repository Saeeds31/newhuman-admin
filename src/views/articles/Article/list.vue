<template>
  <div class="container mt-4" v-if="checkPermission(['article_view'])">

    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-book-half"></i>
          <span>مدیریت مقاله</span>
        </h3>
        <router-link to="/articles/create" class="btn btn-success">

          <i class="bi bi-plus"></i>
          <span>
            افزودن مقاله

          </span>
        </router-link>
      </div>
      <div class="card-body">
        <form @submit.prevent="getArticles()">
          <div class="row g-2">
            <div class="col-md-4">
              <input v-model="filters.title" type="text" class="form-control" placeholder="جستجو بر اساس عنوان" />
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
          <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">در حال بارگذاری...</span>
          </div>
        </div>

        <div v-else>
          <table class="table table-bordered table-striped">
            <thead>
              <tr>
                <th>شناسه</th>
                <th>عنوان</th>
                <th>اسلاگ</th>
                <th>مدت زمان مطالعه</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="article in articles.data" :key="article.id">
                <td>{{ article.id }}</td>
                <td>{{ article.title }}</td>
                <td>{{ article.slug }}</td>
                <td>{{ article.read_time }} </td>
                <td>
                  <router-link :to="`/articles/${article.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>
                  <button class="btn btn-sm btn-danger" @click="deleteArticle(article.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
          <!-- صفحه بندی  -->

          <b-pagination v-model="currentPage" :total-rows="articles.total" v-if="articles.last_page != 1"
            :per-page="articles.per_page" @Update:modelValue="changePage" align="center" class="mt-3"></b-pagination>

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
const currentPage = ref(1)
const articles = ref({ data: [], meta: null });
const loading = ref(false);
const filters = ref({ title: "" });
let currentUrl = "/articles";
async function getArticles(url) {
  loading.value = true;
  try {
    const { data } = await axios.get(url, { params: filters.value });
    articles.value = data.data;
    currentPage.value = data.data.current_page
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

const changePage = (page) => {
  if (page) getArticles(`${currentUrl}?page=${page}`);
  else currentUrl = "/articles"
};

const deleteArticle = (id) => {
  Swal.fire({
    title: "حذف مقاله",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/articles/${id}`);
        Swal.fire("موفق", "مقاله حذف شد", "success");
        getArticles();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getArticles(currentUrl);
});
</script>

<style scoped>
.table th,
.table td {
  vertical-align: middle;
}
</style>