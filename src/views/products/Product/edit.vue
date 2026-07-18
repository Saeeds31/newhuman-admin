<template>
  <div class="container py-4" v-if="checkPermission(['product_update'])">
    <div class="card">
      <div class="card-header">
        <h3><i class="bi bi-pencil-square"></i> ویرایش محصول</h3>
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
                  <label class="form-label">افزودن تصاویر جدید</label>
                  <VueFileAgent
                    @select="imagesLoaded"
                    :maxFiles="10"
                    accept="image/*"
                    theme="grid"
                    deletable
                    sortable
                  />
                  <small class="text-muted">حداکثر ۱۰ تصویر - فرمت‌های مجاز: jpg, png, webp</small>
                  <span v-if="errors.images" class="text-danger d-block">{{ errors.images[0] }}</span>
                </div>

                <!-- نمایش تصاویر موجود -->
                <div class="col-12" v-if="existingImages.length > 0">
                  <label class="form-label">تصاویر موجود (برای حذف روی × کلیک کنید)</label>
                  <div class="row">
                    <div class="col-md-3 mb-2" v-for="(img, index) in existingImages" :key="img.id || index">
                      <div class="position-relative">
                        <img :src="baseImageAddress + img.path" class="img-fluid rounded" style="height:150px;width:100%;object-fit:cover" />
                        <button type="button" class="btn btn-sm btn-danger position-absolute top-0 end-0 m-1" @click="markImageForDeletion(index)">
                          <i class="bi bi-x"></i>
                        </button>
                        <div class="text-center mt-1">
                          <small>{{ img.is_deleted ? 'در انتظار حذف' : 'ترتیب: ' + (index + 1) }}</small>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>

                <!-- نمایش تصاویر جدید آپلود شده -->
                <div class="col-12" v-if="newImages.length > 0">
                  <label class="form-label">تصاویر جدید</label>
                  <div class="row">
                    <div class="col-md-3 mb-2" v-for="(img, index) in newImages" :key="'new-' + index">
                      <div class="position-relative">
                        <img :src="img.url" class="img-fluid rounded" style="height:150px;width:100%;object-fit:cover" />
                        <button type="button" class="btn btn-sm btn-danger position-absolute top-0 end-0 m-1" @click="removeNewImage(index)">
                          <i class="bi bi-x"></i>
                        </button>
                        <div class="text-center mt-1">
                          <small>جدید</small>
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
                  <VueFileAgent
                    @select="videoLoaded"
                    :maxFiles="1"
                    accept="video/*"
                    theme="grid"
                    deletable
                    sortable
                  />
                  <small class="text-muted">فرمت‌های مجاز: mp4, avi, mkv</small>
                  <span v-if="errors.video" class="text-danger d-block">{{ errors.video[0] }}</span>
                </div>

                <div class="col-12" v-if="form.video && !videoDeleted">
                  <video :src="baseImageAddress + form.video" controls style="width:100%;max-height:400px"></video>
                  <button class="btn btn-danger btn-sm mt-2" @click="deleteVideo">
                    <i class="bi bi-trash3"></i> حذف ویدیو
                  </button>
                </div>

                <div class="col-12" v-if="videoDeleted">
                  <div class="alert alert-warning">ویدیو برای حذف علامت‌گذاری شده است</div>
                </div>

                <div class="col-12" v-if="newVideo">
                  <video :src="newVideo" controls style="width:100%;max-height:400px"></video>
                  <div class="alert alert-success mt-2">ویدیو جدید برای آپلود آماده است</div>
                </div>
              </div>
            </div>

            <!-- استپ ۵: دسته‌بندی‌ها -->
            <div v-if="currentStep === 4" class="step-panel">
              <div class="row">
                <div class="col-12 mb-3">
                  <label class="form-label">دسته‌بندی‌ها</label>
                  <Treeselect
                    v-model="form.categories"
                    :multiple="true"
                    :options="categoryOptions"
                    :normalizer="normalizer"
                    placeholder="دسته‌بندی‌ها را انتخاب کنید"
                  />
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

            <!-- استپ ۷: فایل‌های محصول -->
            <div v-if="currentStep === 6" class="step-panel">
              <div class="row">
                <div class="col-12 mb-3">
                  <h5>فایل‌های محصول</h5>
                  <p class="text-muted">مدیریت فایل‌های قابل دانلود این محصول</p>
                  <hr />
                </div>

                <!-- لیست فایل‌های موجود -->
                <div class="col-12" v-if="existingFiles.length > 0">
                  <label class="form-label">فایل‌های موجود</label>
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
                        <tr v-for="(file, index) in existingFiles" :key="file.id || index">
                          <td>{{ index + 1 }}</td>
                          <td>
                            <input v-model="file.title" type="text" class="form-control form-control-sm" placeholder="عنوان فایل" />
                          </td>
                          <td>
                            <input v-model="file.path" type="text" class="form-control form-control-sm" placeholder="آدرس فایل" />
                          </td>
                          <td>
                            <input v-model="file.is_free" type="checkbox" />
                          </td>
                          <td>
                            <button class="btn btn-sm btn-danger" @click="markFileForDeletion(index)">
                              <i class="bi bi-x"></i>
                            </button>
                            <span v-if="file.is_deleted" class="badge bg-warning ms-1">در انتظار حذف</span>
                          </td>
                        </tr>
                      </tbody>
                    </table>
                  </div>
                </div>

                <!-- لیست فایل‌های جدید -->
                <div class="col-12" v-if="newFiles.length > 0">
                  <label class="form-label">فایل‌های جدید</label>
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
                        <tr v-for="(file, index) in newFiles" :key="'new-' + index">
                          <td>{{ index + 1 }}</td>
                          <td>
                            <input v-model="file.title" type="text" class="form-control form-control-sm" placeholder="عنوان فایل" />
                          </td>
                          <td>
                            <input v-model="file.path" type="text" class="form-control form-control-sm" placeholder="آدرس فایل" />
                          </td>
                          <td>
                            <input v-model="file.is_free" type="checkbox" />
                          </td>
                          <td>
                            <button class="btn btn-sm btn-danger" @click="removeNewFile(index)">
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
                        <input v-model="newFile.title" type="text" class="form-control" placeholder="عنوان فایل (اختیاری)" />
                      </div>
                      <div class="col-md-5 mb-2">
                        <input v-model="newFile.path" type="text" class="form-control" placeholder="آدرس فایل را وارد کنید" />
                      </div>
                      <div class="col-md-1 mb-2">
                        <div class="form-check mt-2">
                          <input v-model="newFile.is_free" type="checkbox" class="form-check-input" id="edit_file_is_free" />
                          <label class="form-check-label" for="edit_file_is_free">رایگان</label>
                        </div>
                      </div>
                      <div class="col-md-2 mb-2">
                        <button class="btn btn-success w-100" @click="addNewFile" :disabled="!newFile.path">
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
              <i class="bi bi-save2"></i> بروزرسانی
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
import { useRoute, useRouter } from 'vue-router';
import { useAdmin } from '@/stores/modules/admin';
import Treeselect from 'vue3-treeselect';
import 'vue3-treeselect/dist/vue3-treeselect.css';
import Editor from '@/components/shared/editor.vue';

const store = useAdmin();
const checkPermission = store.checkPermission;
const route = useRoute();
const router = useRouter();
const productId = route.params.id;
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
  { label: 'فایل‌ها', icon: 'bi-file-earmark' }
]);

const productTypes = ref([]);
const categoryOptions = ref([]);
const attributes = ref([]);
const errors = ref({});
const existingImages = ref([]);
const newImages = ref([]);
const existingFiles = ref([]);
const newFiles = ref([]);
const newVideo = ref(null);
const videoDeleted = ref(false);

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
  video: null
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

async function loadProduct() {
  try {
    const { data } = await axios.get(`/products/${productId}`);
    const product = data.data;
    
    form.value = {
      product_type_id: product.product_type_id,
      title: product.title,
      description: product.description,
      status: product.status,
      price: product.price,
      discount_value: product.discount_value,
      discount_type: product.discount_type,
      is_free: product.is_free,
      meta_title: product.meta_title,
      meta_description: product.meta_description,
      categories: product.categories.map(c => c.id),
      attributes: {},
      video: product.video
    };

    // بارگذاری تصاویر
    existingImages.value = product.images.map(img => ({
      ...img,
      is_deleted: false
    })) || [];

    // بارگذاری فایل‌ها
    existingFiles.value = product.files.map(file => ({
      ...file,
      is_deleted: false
    })) || [];

    // بارگذاری ویژگی‌ها
    await loadAttributes();
    
    // مقداردهی ویژگی‌ها
    if (product.attribute_values) {
      product.attribute_values.forEach(attr => {
        form.value.attributes[attr.product_attribute_id] = attr.value;
      });
    }
  } catch (err) {
    toast.error('خطا در بارگذاری محصول');
    router.push('/products');
  }
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
  } catch (err) {
    console.error(err);
  }
}

// مدیریت تصاویر
function imagesLoaded(files) {
  for (const file of files) {
    const reader = new FileReader();
    reader.onload = (e) => {
      newImages.value.push({
        file: file.file,
        url: e.target.result,
        name: file.file.name
      });
    };
    reader.readAsDataURL(file.file);
  }
}

function markImageForDeletion(index) {
  const image = existingImages.value[index];
  if (image.id) {
    image.is_deleted = !image.is_deleted;
  }
}

function removeNewImage(index) {
  newImages.value.splice(index, 1);
}

// مدیریت فایل‌ها
function markFileForDeletion(index) {
  const file = existingFiles.value[index];
  if (file.id) {
    file.is_deleted = !file.is_deleted;
  }
}

function addNewFile() {
  if (!newFile.value.path) {
    toast.warning('لطفا آدرس فایل را وارد کنید');
    return;
  }

  newFiles.value.push({
    title: newFile.value.title || newFile.value.path.split('/').pop() || 'فایل بدون عنوان',
    description: '',
    path: newFile.value.path,
    is_free: newFile.value.is_free,
    sort_order: existingFiles.value.length + newFiles.value.length
  });

  newFile.value = {
    title: '',
    path: '',
    is_free: false
  };

  toast.success('فایل اضافه شد');
}

function removeNewFile(index) {
  newFiles.value.splice(index, 1);
}

// مدیریت ویدیو
function videoLoaded(files) {
  if (files.length === 0) return;
  const file = files[0].file;
  const reader = new FileReader();
  reader.onload = (e) => {
    newVideo.value = e.target.result;
    form.value.video = file;
  };
  reader.readAsDataURL(file);
}

function deleteVideo() {
  videoDeleted.value = true;
  form.value.video = null;
  newVideo.value = null;
  toast.info('ویدیو برای حذف علامت‌گذاری شد');
}

// ذخیره نهایی
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
    
    // تصاویر حذف شده
    const deletedImageIds = existingImages.value
      .filter(img => img.is_deleted && img.id)
      .map(img => img.id);
    
    if (deletedImageIds.length > 0) {
      formData.append('deleted_images', JSON.stringify(deletedImageIds));
    }
    
    // تصاویر جدید
    if (newImages.value.length > 0) {
      for (const img of newImages.value) {
        formData.append('images[]', img.file);
      }
    }
    
    // فایل‌های موجود (ویرایش شده)
    const updatedFiles = existingFiles.value
      .filter(file => !file.is_deleted)
      .map(file => ({
        id: file.id,
        title: file.title,
        path: file.path,
        is_free: file.is_free ? 1 : 0,
        sort_order: file.sort_order || 0
      }));

    if (updatedFiles.length > 0) {
      formData.append('updated_files', JSON.stringify(updatedFiles));
    }

    // فایل‌های حذف شده
    const deletedFileIds = existingFiles.value
      .filter(file => file.is_deleted && file.id)
      .map(file => file.id);

    if (deletedFileIds.length > 0) {
      formData.append('deleted_files', JSON.stringify(deletedFileIds));
    }

    // فایل‌های جدید
    if (newFiles.value.length > 0) {
      for (const [index, file] of newFiles.value.entries()) {
        formData.append(`new_files[${index}][title]`, file.title || '');
        formData.append(`new_files[${index}][path]`, file.path);
        formData.append(`new_files[${index}][is_free]`, file.is_free ? 1 : 0);
        formData.append(`new_files[${index}][sort_order]`, existingFiles.value.length + index);
      }
    }
    
    // ویدیو
    if (form.value.video && typeof form.value.video === 'object') {
      formData.append('video', form.value.video);
    }
    
    // حذف ویدیو
    if (videoDeleted.value) {
      formData.append('delete_video', '1');
    }

    const { data } = await axios.post(`/products/${productId}`, formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
      params: { _method: 'PUT' }
    });
    
    toast.success('محصول با موفقیت بروزرسانی شد!');
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
      } else if (errors.value.attributes) {
        currentStep.value = 5;
      } else if (errors.value.product_files || errors.value.deleted_files) {
        currentStep.value = 6;
      }
    }
    toast.error('خطا در بروزرسانی محصول');
  } finally {
    loading.value = false;
  }
}

onMounted(() => {
  loadProductTypes();
  loadCategories();
  loadProduct();
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