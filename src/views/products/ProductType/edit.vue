<template>
    <div class="container py-4" v-if="checkPermission(['producttype_update'])">
      <div class="card">
        <div class="card-header">
          <h3>
            <i class="bi bi-pencil-square"></i>
            <span>ویرایش دسته‌بندی</span>
          </h3>
        </div>
        <div class="card-body">
          <form @submit.prevent="submitData">
            <div class="row">
              <div class="col-md-6 mb-3">
                <label class="form-label">نام دسته‌بندی</label>
                <input v-model="form.name" type="text" class="form-control" />
                <span v-if="errors.name" class="text-danger">{{ errors.name[0] }}</span>
              </div>
  
              <div class="col-md-6 mb-3">
                <label class="form-label">slug</label>
                <input v-model="form.slug" type="text" class="form-control" />
                <span v-if="errors.slug" class="text-danger">{{ errors.slug[0] }}</span>
              </div>
  
              <div class="col-md-12 mb-3">
                <label class="form-label">عنوان متا</label>
                <input v-model="form.meta_title" type="text" class="form-control" />
              </div>
  
              <div class="col-md-12 mb-3">
                <label class="form-label">توضیحات متا</label>
                <textarea v-model="form.meta_description" class="form-control" rows="3"></textarea>
              </div>
  
              <div class="col-md-6 mb-3">
                <div class="form-check">
                  <input v-model="form.is_active" type="checkbox" class="form-check-input" id="is_active" />
                  <label class="form-check-label" for="is_active">فعال</label>
                </div>
              </div>
  
              <div class="col-md-6 mb-3">
                <label class="form-label">ترتیب</label>
                <input v-model.number="form.sort_order" type="number" class="form-control" min="0" />
              </div>
            </div>
  
            <button :disabled="loading" type="submit" class="btn btn-primary mt-3">
              <i class="bi bi-save2"></i>
              <span class="mx-2">بروزرسانی</span>
            </button>
          </form>
        </div>
      </div>
    </div>
  </template>
  
  <script setup>
  import { ref, onMounted } from 'vue';
  import axios from 'axios';
  import { toast } from 'vue3-toastify';
  import { useRoute, useRouter } from 'vue-router';
  import { useAdmin } from '@/stores/modules/admin';

const store = useAdmin();
const checkPermission = store.checkPermission;
  const route = useRoute();
  const router = useRouter();
  const id = route.params.id;
  const loading = ref(false);
  const form = ref({
    name: '',
    slug: '',
    meta_title: '',
    meta_description: '',
    is_active: true,
    sort_order: 0
  });
  const errors = ref({});
  
  async function loadItem() {
    try {
      const { data } = await axios.get(`/product-types/${id}`);
      Object.assign(form.value, data.data);
    } catch (e) {
      toast.error('خطا در بارگذاری');
    }
  }
  
  async function submitData() {
    errors.value = {};
    loading.value = true;
    try {
      await axios.put(`/product-types/${id}`, form.value);
      toast.success('دسته‌بندی با موفقیت بروزرسانی شد!');
      router.push('/products/product-types');
    } catch (e) {
      if (e.response?.data?.errors) {
        errors.value = e.response.data.errors;
      }
      toast.error('خطا در بروزرسانی');
    } finally {
      loading.value = false;
    }
  }
  
  onMounted(() => {
    loadItem();
  });
  </script>