<template>
  <div class="container py-4" v-if="checkPermission(['product_store'])">
    <div class="card">
      <div class="card-header">
        <h3><i class="bi bi-plus-circle"></i> ایجاد محصول جدید</h3>
      </div>
      <div class="card-body">
        <!-- استپ‌های سفارشی -->
        <div class="steps-wrapper">
          <div class="steps-header">
            <div v-for="(step, index) in steps" :key="index" class="step-item"
              :class="{ active: currentStep === index, completed: currentStep > index }">
              <div class="step-circle" @click="goToStep(index)">
                <i :class="step.icon"></i>
                <span class="step-number">{{ index + 1 }}</span>
              </div>
              <div class="step-label">{{ step.label }}</div>
            </div>
          </div>

          <div class="step-content">
            <!-- استپ ۱: اطلاعات اصلی -->
            <div v-if="currentStep === 0" class="step-panel">
              <div class="row">
                <div class="col-md-6 mb-3">
                  <label class="form-label">نوع محصول <span class="text-danger">*</span></label>
                  <select v-model="form.product_type_id" @change="loadAttributes" class="form-control" required>
                    <option value="">انتخاب کنید</option>
                    <option v-for="type in productTypes" :key="type.id" :value="type.id">{{ type.name }}</option>
                  </select>
                  <span v-if="errors.product_type_id" class="text-danger">{{ errors.product_type_id[0] }}</span>
                </div>

                <div class="col-md-6 mb-3">
                  <label class="form-label">عنوان <span class="text-danger">*</span></label>
                  <input v-model="form.title" type="text" class="form-control" />
                  <span v-if="errors.title" class="text-danger">{{ errors.title[0] }}</span>
                </div>

                <div class="col-md-12 mb-3">
                  <label class="form-label">توضیحات</label>
                  <Editor v-model="form.description" />
                  <span v-if="errors.description" class="text-danger">{{ errors.description[0] }}</span>
                </div>

                <div class="col-md-6 mb-3">
                  <label class="form-label">وضعیت</label>
                  <select v-model="form.status" class="form-control">
                    <option value="draft">پیش‌نویس</option>
                    <option value="published">منتشر شده</option>
                    <option value="unpublished">منتشر نشده</option>
                  </select>
                </div>

                <div class="col-md-6 mb-3">
                  <label class="form-label">عنوان متا</label>
                  <input v-model="form.meta_title" type="text" class="form-control" />
                </div>

                <div class="col-md-12 mb-3">
                  <label class="form-label">توضیحات متا</label>
                  <textarea v-model="form.meta_description" class="form-control" rows="2"></textarea>
                </div>
              </div>
            </div>

            <!-- استپ ۲: قیمت و تخفیف -->
            <div v-if="currentStep === 1" class="step-panel">
              <div class="row">
                <div class="col-md-6 mb-3">
                  <label class="form-label">قیمت (تومان)</label>
                  <input v-model.number="form.price" type="number" class="form-control" min="0" />
                  <span v-if="errors.price" class="text-danger">{{ errors.price[0] }}</span>
                </div>

                <div class="col-md-3 mb-3">
                  <label class="form-label">مقدار تخفیف</label>
                  <input v-model.number="form.discount_value" type="number" class="form-control" min="0" />
                </div>

                <div class="col-md-3 mb-3">
                  <label class="form-label">نوع تخفیف</label>
                  <select v-model="form.discount_type" class="form-control">
                    <option value="">بدون تخفیف</option>
                    <option value="percent">درصدی</option>
                    <option value="fixed">ثابت</option>
                  </select>
                </div>

                <div class="col-md-6 mb-3">
                  <div class="form-check">
                    <input v-model="form.is_free" type="checkbox" class="form-check-input" id="is_free" />
                    <label class="form-check-label" for="is_free">محصول رایگان</label>
                  </div>
                </div>

                <div class="col-md-6 mb-3" v-if="!form.is_free && form.price > 0">
                  <label class="form-label">قیمت نهایی</label>
                  <input :value="numberFormat(finalPrice)" type="text" class="form-control" disabled />
                </div>
              </div>
            </div>

            <!-- استپ ۳: تصاویر -->
            <div v-if="currentStep === 2" class="step-panel">
              <div class="row">
                <div class="col-12 mb-3">
                  <label class="form-label">تصاویر محصول</label>
                  <VueFileAgent @select="imagesLoaded" :maxFiles="10" accept="image/*" theme="grid" deletable
                    sortable />
                  <small class="text-muted">حداکثر ۱۰ تصویر - فرمت‌های مجاز: jpg, png, webp</small>
                  <span v-if="errors.images" class="text-danger d-block">{{ errors.images[0] }}</span>
                </div>

                <div class="col-12" v-if="uploadedImages.length > 0">
                  <label class="form-label">تصاویر آپلود شده</label>
                  <div class="row">
                    <div class="col-md-3 mb-2" v-for="(img, index) in uploadedImages" :key="index">
                      <div class="position-relative">
                        <img :src="img.url" class="img-fluid rounded"
                          style="height:150px;width:100%;object-fit:cover" />
                        <button type="button" class="btn btn-sm btn-danger position-absolute top-0 end-0 m-1"
                          @click="removeImage(index)">
                          <i class="bi bi-x"></i>
                        </button>
                        <div class="text-center mt-1">
                          <small>ترتیب: {{ index + 1 }}</small>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </div>
            </div>

            <!-- استپ ۴: ویدیو -->
            <div v-if="currentStep === 3" class="step-panel">
              <div class="row">
                <div class="col-12 mb-3">
                  <label class="form-label">ویدیو</label>
                  <VueFileAgent @select="videoLoaded" :maxFiles="1" accept="video/*" theme="grid" deletable sortable />
                  <small class="text-muted">فرمت‌های مجاز: mp4, avi, mkv</small>
                  <span v-if="errors.video" class="text-danger d-block">{{ errors.video[0] }}</span>
                </div>

                <div class="col-12" v-if="uploadedVideo">
                  <video :src="uploadedVideo" controls style="width:100%;max-height:400px"></video>
                </div>
              </div>
            </div>

            <!-- استپ ۵: دسته‌بندی‌ها -->
            <div v-if="currentStep === 4" class="step-panel">
              <div class="row">
                <div class="col-12 mb-3">
                  <label class="form-label">دسته‌بندی‌ها</label>
                  <Treeselect v-model="form.categories" :multiple="true" :options="categoryOptions"
                    :normalizer="normalizer" placeholder="دسته‌بندی‌ها را انتخاب کنید" />
                  <span v-if="errors.categories" class="text-danger">{{ errors.categories[0] }}</span>
                </div>
              </div>
            </div>

            <!-- استپ ۶: ویژگی‌ها -->
            <div v-if="currentStep === 5" class="step-panel">
              <div class="row">
                <div class="col-12 mb-3" v-if="attributes.length === 0">
                  <div class="alert alert-info">برای این نوع محصول ویژگی‌ای تعریف نشده است.</div>
                </div>
                <div class="col-md-6 mb-3" v-for="attr in attributes" :key="attr.id">
                  <label class="form-label">
                    {{ attr.name }}
                    <span class="text-danger" v-if="attr.is_required">*</span>
                  </label>
                  <input v-model="form.attributes[attr.id]" type="text" class="form-control" />
                  <span v-if="errors['attributes.' + attr.id]" class="text-danger">
                    {{ errors['attributes.' + attr.id][0] }}
                  </span>
                </div>
              </div>
            </div>
            <!-- استپ ۷: فایل‌های محصول (جدید) -->
            <div v-if="currentStep === 6" class="step-panel">
              <div class="row">
                <div class="col-12 mb-3">
                  <h5>فایل‌های محصول</h5>
                  <p class="text-muted">آدرس فایل‌های قابل دانلود برای این محصول را وارد کنید</p>
                  <hr />
                </div>

                <!-- لیست فایل‌های اضافه شده -->
                <div class="col-12" v-if="productFiles.length > 0">
                  <div class="table-responsive">
                    <table class="table table-bordered">
                      <thead>
                        <tr>
                          <th>#</th>
                          <th>عنوان</th>
                          <th>آدرس فایل</th>
                          <th>رایگان</th>
                          <th>عملیات</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="(file, index) in productFiles" :key="index">
                          <td>{{ index + 1 }}</td>
                          <td>
                            <input v-model="file.title" type="text" class="form-control form-control-sm"
                              placeholder="عنوان فایل" />
                          </td>
                          <td>
                            <input v-model="file.path" type="text" class="form-control form-control-sm"
                              placeholder="آدرس فایل" />
                          </td>
                          <td>
                            <input v-model="file.is_free" type="checkbox" />
                          </td>
                          <td>
                            <button class="btn btn-sm btn-danger" @click="removeProductFile(index)">
                              <i class="bi bi-trash3"></i>
                            </button>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>

                <!-- افزودن فایل جدید -->
                <div class="col-12">
                  <div class="border p-3 rounded">
                    <h6>افزودن فایل جدید</h6>
                    <div class="row">
                      <div class="col-md-4 mb-2">
                        <input v-model="newFile.title" type="text" class="form-control"
                          placeholder="عنوان فایل (اختیاری)" />
                      </div>
                      <div class="col-md-5 mb-2">
                        <input v-model="newFile.path" type="text" class="form-control"
                          placeholder="آدرس فایل را وارد کنید" />
                      </div>
                      <div class="col-md-1 mb-2">
                        <div class="form-check mt-2">
                          <input v-model="newFile.is_free" type="checkbox" class="form-check-input" id="file_is_free" />
                          <label class="form-check-label" for="file_is_free">رایگان</label>
                        </div>
                      </div>
                      <div class="col-md-2 mb-2">
                        <button class="btn btn-success w-100" @click="addProductFile" :disabled="!newFile.path">
                          <i class="bi bi-plus"></i> افزودن
                        </button>
                      </div>
                    </div>
                    <small class="text-muted">آدرس فایل را از بخش مدیریت فایل‌ها کپی کنید</small>
                    <span v-if="errors.product_files" class="text-danger d-block">{{ errors.product_files[0] }}</span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>

        <!-- دکمه‌های ناوبری -->
        <div class="d-flex justify-content-between mt-4">
          <button class="btn btn-secondary" @click="prevStep" :disabled="currentStep === 0">
            <i class="bi bi-arrow-right"></i> قبلی
          </button>
          <div>
            <button class="btn btn-danger me-2" @click="cancelForm">
              <i class="bi bi-x"></i> انصراف
            </button>
            <button v-if="currentStep < steps.length - 1" class="btn btn-primary" @click="nextStep">
              بعدی <i class="bi bi-arrow-left"></i>
            </button>
            <button v-else class="btn btn-success" :disabled="loading" @click="submitForm">
              <i class="bi bi-save2"></i> ذخیره نهایی
            </button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from 'vue';
import axios from 'axios';
import { toast } from 'vue3-toastify';
import { useRouter } from 'vue-router';
import { useAdmin } from '@/stores/modules/admin';
import Treeselect from 'vue3-treeselect';
import 'vue3-treeselect/dist/vue3-treeselect.css';
import Editor from '@/components/shared/editor.vue';

const store = useAdmin();
const checkPermission = store.checkPermission;
const router = useRouter();
const baseImageAddress = window.baseImageAddress;

const loading = ref(false);
const currentStep = ref(0);
const steps = ref([
  { label: 'اطلاعات اصلی', icon: 'bi-info-circle' },
  { label: 'قیمت', icon: 'bi-coin' },
  { label: 'تصاویر', icon: 'bi-images' },
  { label: 'ویدیو', icon: 'bi-play-circle' },
  { label: 'دسته‌بندی', icon: 'bi-tags' },
  { label: 'ویژگی‌ها', icon: 'bi-list-ul' },
  { label: 'فایل‌ها', icon: 'bi-file-earmark' } // استپ جدید
]);

const productTypes = ref([]);
const categoryOptions = ref([]);
const attributes = ref([]);
const errors = ref({});
const uploadedImages = ref([]);
const uploadedVideo = ref(null);
const productFiles = ref([]);

// فرم فایل جدید
const newFile = ref({
  title: '',
  path: '',
  is_free: false
});

const form = ref({
  product_type_id: '',
  title: '',
  description: '',
  status: 'draft',
  price: 0,
  discount_value: null,
  discount_type: null,
  is_free: false,
  meta_title: '',
  meta_description: '',
  categories: [],
  attributes: {},
  images: [],
  video: null,
  product_files: [] // فایل‌های محصول
});

const normalizer = node => ({ id: node.id, label: node.title, children: node.all_children });

const finalPrice = computed(() => {
  if (form.value.is_free) return 0;
  let price = form.value.price || 0;
  if (form.value.discount_value && form.value.discount_value > 0) {
    if (form.value.discount_type === 'percent') {
      price = price - (price * form.value.discount_value / 100);
    } else {
      price = price - form.value.discount_value;
    }
    return Math.max(0, price);
  }
  return price;
});

function numberFormat(value) {
  if (!value && value !== 0) return '۰';
  return new Intl.NumberFormat('fa-IR').format(value);
}

function formatFileSize(bytes) {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
}

function goToStep(index) {
  if (index <= currentStep.value) {
    currentStep.value = index;
  }
}

function nextStep() {
  if (currentStep.value < steps.value.length - 1) {
    currentStep.value++;
  }
}

function prevStep() {
  if (currentStep.value > 0) {
    currentStep.value--;
  }
}

function cancelForm() {
  router.push('/products');
}

async function loadProductTypes() {
  try {
    const { data } = await axios.get('/product-types');
    productTypes.value = data.data.data;
  } catch (err) {
    console.error(err);
  }
}

async function loadCategories() {
  try {
    const { data } = await axios.get('/categories');
    categoryOptions.value = data.data;
  } catch (err) {
    console.error(err);
  }
}

async function loadAttributes() {
  if (!form.value.product_type_id) {
    attributes.value = [];
    return;
  }
  try {
    const { data } = await axios.get(`/product-types/${form.value.product_type_id}/attributes`);
    attributes.value = data.data;
    attributes.value.forEach(attr => {
      if (!form.value.attributes[attr.id]) {
        form.value.attributes[attr.id] = '';
      }
    });
  } catch (err) {
    console.error(err);
  }
}

function imagesLoaded(files) {
  for (const file of files) {
    const reader = new FileReader();
    reader.onload = (e) => {
      uploadedImages.value.push({
        file: file.file,
        url: e.target.result,
        name: file.file.name
      });
    };
    reader.readAsDataURL(file.file);
  }
}

function removeImage(index) {
  uploadedImages.value.splice(index, 1);
}

function videoLoaded(files) {
  if (files.length === 0) return;
  const file = files[0].file;
  const reader = new FileReader();
  reader.onload = (e) => {
    uploadedVideo.value = e.target.result;
    form.value.video = file;
  };
  reader.readAsDataURL(file);
}

// مدیریت فایل‌های محصول
function handleFileSelect(event) {
  const file = event.target.files[0];
  if (file) {
    newFile.value.file = file;
    // اگر عنوان وارد نشده، از نام فایل استفاده کن
    if (!newFile.value.title) {
      newFile.value.title = file.name.replace(/\.[^/.]+$/, '');
    }
  }
}


function addProductFile() {
  if (!newFile.value.path) {
    toast.warning('لطفا آدرس فایل را وارد کنید');
    return;
  }

  productFiles.value.push({
    title: newFile.value.title || newFile.value.path.split('/').pop() || 'فایل بدون عنوان',
    description: '',
    path: newFile.value.path, // فقط آدرس
    is_free: newFile.value.is_free,
    sort_order: productFiles.value.length
  });

  // reset
  newFile.value = {
    title: '',
    path: '',
    is_free: false
  };

  toast.success('فایل اضافه شد');
}
function removeProductFile(index) {
  productFiles.value.splice(index, 1);
}


async function submitForm() {
  errors.value = {};
  loading.value = true;

  try {
    const formData = new FormData();

    // اطلاعات پایه
    formData.append('product_type_id', form.value.product_type_id);
    formData.append('title', form.value.title);
    formData.append('description', form.value.description || '');
    formData.append('status', form.value.status);
    formData.append('price', form.value.price || 0);
    formData.append('discount_value', form.value.discount_value || '');
    formData.append('discount_type', form.value.discount_type || '');
    formData.append('is_free', form.value.is_free ? 1 : 0);
    formData.append('meta_title', form.value.meta_title || '');
    formData.append('meta_description', form.value.meta_description || '');

    // دسته‌بندی‌ها
    formData.append('categories', JSON.stringify(form.value.categories));

    // ویژگی‌ها
    formData.append('attributes', JSON.stringify(form.value.attributes));

    // تصاویر
    if (uploadedImages.value.length > 0) {
      for (const img of uploadedImages.value) {
        formData.append('images[]', img.file);
      }
    }

    // ویدیو
    if (form.value.video) {
      formData.append('video', form.value.video);
    }

    // فایل‌های محصول - فقط آدرس
    if (productFiles.value.length > 0) {
      for (const [index, file] of productFiles.value.entries()) {
        formData.append(`product_files[${index}][title]`, file.title || '');
        formData.append(`product_files[${index}][description]`, file.description || '');
        formData.append(`product_files[${index}][path]`, file.path);
        formData.append(`product_files[${index}][is_free]`, file.is_free ? 1 : 0);
        formData.append(`product_files[${index}][sort_order]`, index);
      }
    }

    const { data } = await axios.post('/products', formData, {
      headers: { 'Content-Type': 'multipart/form-data' }
    });

    toast.success('محصول با موفقیت ایجاد شد!');
    router.push('/products');
  } catch (e) {
    if (e.response?.data?.errors) {
      errors.value = e.response.data.errors;
      if (errors.value.title || errors.value.product_type_id) {
        currentStep.value = 0;
      } else if (errors.value.price) {
        currentStep.value = 1;
      } else if (errors.value.images) {
        currentStep.value = 2;
      } else if (errors.value.categories) {
        currentStep.value = 4;
      } else if (errors.value.product_files) {
        currentStep.value = 6;
      }
    }
    toast.error('خطا در ذخیره محصول');
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  loadProductTypes();
  loadCategories();
});
</script>

<style scoped>
.steps-wrapper {
  background: #f8f9fa;
  border-radius: 8px;
  padding: 20px;
}

.steps-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  position: relative;
}

.steps-header::before {
  content: '';
  position: absolute;
  top: 25px;
  left: 40px;
  right: 40px;
  height: 3px;
  background: #dee2e6;
  z-index: 1;
}

.step-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  z-index: 2;
}

.step-circle {
  width: 50px;
  height: 50px;
  border-radius: 50%;
  background: #dee2e6;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  position: relative;
  transition: all 0.3s;
}

.step-item.active .step-circle {
  background: #0d6efd;
  color: white;
  box-shadow: 0 0 0 5px rgba(13, 110, 253, 0.2);
}

.step-item.completed .step-circle {
  background: #198754;
  color: white;
}

.step-circle .step-number {
  display: none;
}

.step-label {
  margin-top: 8px;
  font-size: 12px;
  font-weight: 500;
  color: #6c757d;
  text-align: center;
}

.step-item.active .step-label {
  color: #0d6efd;
  font-weight: 600;
}

.step-item.completed .step-label {
  color: #198754;
}

.step-content {
  background: white;
  border-radius: 8px;
  padding: 20px;
  min-height: 300px;
}

.step-panel {
  animation: fadeIn 0.3s;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}

@media (max-width: 768px) {
  .steps-header {
    flex-wrap: wrap;
    gap: 10px;
  }

  .steps-header::before {
    display: none;
  }

  .step-circle {
    width: 40px;
    height: 40px;
    font-size: 14px;
  }

  .step-label {
    font-size: 10px;
  }
}
</style>