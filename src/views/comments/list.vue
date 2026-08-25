<template>
  <div class="container mt-4" v-if="checkPermission(['comment_view'])">

    <!-- کارت فیلترها -->
    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-chat-dots"></i>
          <span>مدیریت کامنت‌ها</span>
        </h3>
        <div>
          <span class="badge bg-info me-2">در انتظار: {{ stats.pending }}</span>
          <span class="badge bg-success me-2">تایید شده: {{ stats.approved }}</span>
          <span class="badge bg-danger me-2">رد شده: {{ stats.rejected }}</span>
          <span class="badge bg-secondary">کل: {{ stats.total_comments }}</span>
        </div>
      </div>
      <div class="card-body">
        <form @submit.prevent="getComments()">
          <div class="row g-2">
            <div class="col-md-3">
              <input v-model="filters.search" type="text" class="form-control" placeholder="جستجو در محتوا..." />
            </div>
            <div class="col-md-2">
              <select v-model="filters.status" class="form-select">
                <option value="">همه وضعیت‌ها</option>
                <option value="0">در انتظار</option>
                <option value="1">تایید شده</option>
                <option value="2">رد شده</option>
              </select>
            </div>
            <div class="col-md-2">
              <select v-model="filters.type" class="form-select">
                <option value="">همه انواع</option>
                <option value="Modules\\Articles\\Models\\Article">مقاله</option>
                <option value="Modules\\Products\\Models\\Product">محصول</option>
              </select>
            </div>
            <div class="col-md-3">
              <input v-model="filters.date_from" type="date" class="form-control" placeholder="از تاریخ" />
            </div>
            <div class="col-md-2">
              <button class="btn btn-primary w-100" type="submit">
                <i class="bi bi-search"></i> جستجو
              </button>
            </div>
          </div>
        </form>
      </div>
    </div>

    <!-- جدول کامنت‌ها -->
    <div class="card">
      <div class="card-body">
        <div v-if="loading" class="text-center py-5">
          <div class="spinner-border text-primary" role="status">
            <span class="visually-hidden">در حال بارگذاری...</span>
          </div>
        </div>

        <div v-else>
          <div class="table-responsive">
            <table class="table table-bordered table-striped table-hover">
              <thead>
                <tr>
                  <th style="width: 60px;">#</th>
                  <th>محتوا</th>
                  <th style="width: 130px;">کاربر</th>
                  <th style="width: 150px;">نوع</th>
                  <th style="width: 80px;">امتیاز</th>
                  <th style="width: 100px;">وضعیت</th>
                  <th style="width: 140px;">تاریخ</th>
                  <th style="width: 200px;">عملیات</th>
                </tr>
              </thead>
              <tbody>
                <tr v-for="comment in comments.data" :key="comment.id">
                  <td>{{ comment.id }}</td>
                  <td>
                    <div class="comment-content">
                      <div>{{ truncateText(comment.content, 80) }}</div>
                      <small v-if="comment.replies_count > 0" class="text-muted">
                        <i class="bi bi-reply"></i> {{ comment.replies_count }} پاسخ
                      </small>
                    </div>
                  </td>
                  <td>{{ comment.user?.full_name || 'ناشناس' }}</td>
                  <td>
                    <span class="badge" :class="getTypeBadgeClass(comment.commentable_type)">
                      {{ getTypeLabel(comment.commentable_type) }}
                    </span>
                    <div v-if="comment.commentable" class="small text-muted">
                      ID: {{ comment.commentable_id }}
                    </div>
                  </td>
                  <td>
                    <span v-if="comment.rating" class="badge bg-warning text-dark">
                      <i class="bi bi-star-fill"></i> {{ comment.rating }}
                    </span>
                    <span v-else class="text-muted">-</span>
                  </td>
                  <td>
                    <span class="badge" :class="getStatusBadgeClass(comment.status)">
                      {{ getStatusLabel(comment.status) }}
                    </span>
                  </td>
                  <td>
                    <div class="small">{{ formatDate(comment.created_at) }}</div>
                    <div class="small text-muted">{{ formatTime(comment.created_at) }}</div>
                  </td>
                  <td>
                    <div class="btn-group btn-group-sm" role="group">
                      <button v-if="comment.status === 0 || comment.status === 2" 
                              class="btn btn-success" 
                              @click="changeStatus(comment.id, 1)"
                              title="تایید">
                        <i class="bi bi-check-lg"></i>
                      </button>
                      <button v-if="comment.status === 0 || comment.status === 1" 
                              class="btn btn-danger" 
                              @click="changeStatus(comment.id, 2)"
                              title="رد">
                        <i class="bi bi-x-lg"></i>
                      </button>
                      <button v-if="comment.status !== 0" 
                              class="btn btn-warning" 
                              @click="changeStatus(comment.id, 0)"
                              title="برگشت به در انتظار">
                        <i class="bi bi-arrow-counterclockwise"></i>
                      </button>
                      <button class="btn btn-info" @click="showReplyModal(comment)" title="پاسخ">
                        <i class="bi bi-reply"></i>
                      </button>
                      <button class="btn btn-danger" @click="deleteComment(comment.id)" title="حذف">
                        <i class="bi bi-trash3-fill"></i>
                      </button>
                    </div>
                  </td>
                </tr>
                <tr v-if="!comments.data || comments.data.length === 0">
                  <td colspan="8" class="text-center py-4">
                    <i class="bi bi-inbox"></i> هیچ کامنتی یافت نشد
                  </td>
                </tr>
              </tbody>
            </table>
          </div>

          <!-- صفحه‌بندی -->
          <b-pagination 
            v-model="currentPage" 
            :total-rows="comments.total" 
            v-if="comments.last_page && comments.last_page > 1"
            :per-page="comments.per_page" 
            @update:modelValue="changePage" 
            align="center" 
            class="mt-3">
          </b-pagination>
        </div>
      </div>
    </div>

    <!-- مودال پاسخ به کامنت -->
    <div class="modal fade" id="replyModal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">پاسخ به کامنت</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>
          <div class="modal-body">
            <div class="mb-3">
              <label class="form-label">کامنت اصلی</label>
              <div class="p-2 bg-light rounded">{{ replyTarget?.content }}</div>
            </div>
            <div class="mb-3">
              <label class="form-label">پاسخ</label>
              <textarea v-model="replyContent" class="form-control" rows="4" placeholder="متن پاسخ خود را وارد کنید..."></textarea>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">انصراف</button>
            <button type="button" class="btn btn-primary" @click="submitReply">ارسال پاسخ</button>
          </div>
        </div>
      </div>
    </div>

    <!-- مودال مشاهده کامنت -->
    <div class="modal fade" id="viewModal" tabindex="-1">
      <div class="modal-dialog modal-lg">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">جزئیات کامنت</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>
          <div class="modal-body">
            <div v-if="viewComment">
              <div class="row">
                <div class="col-md-6">
                  <strong>کاربر:</strong> {{ viewComment.user?.name || 'ناشناس' }}
                </div>
                <div class="col-md-6">
                  <strong>وضعیت:</strong> 
                  <span class="badge" :class="getStatusBadgeClass(viewComment.status)">
                    {{ getStatusLabel(viewComment.status) }}
                  </span>
                </div>
              </div>
              <div class="row mt-2">
                <div class="col-md-6">
                  <strong>نوع:</strong> {{ getTypeLabel(viewComment.commentable_type) }}
                </div>
                <div class="col-md-6">
                  <strong>آی‌پی:</strong> {{ viewComment.ip || '-' }}
                </div>
              </div>
              <div class="mt-3">
                <strong>محتوا:</strong>
                <div class="p-2 bg-light rounded">{{ viewComment.content }}</div>
              </div>
              <div v-if="viewComment.rating" class="mt-2">
                <strong>امتیاز:</strong> 
                <span class="text-warning">
                  <i class="bi bi-star-fill" v-for="n in viewComment.rating" :key="n"></i>
                </span>
              </div>
              <div v-if="viewComment.replies && viewComment.replies.length" class="mt-3">
                <strong>پاسخ‌ها:</strong>
                <div v-for="reply in viewComment.replies" :key="reply.id" class="p-2 bg-light rounded mt-1">
                  {{ reply.content }}
                  <small class="text-muted d-block">- {{ reply.user?.name || 'ناشناس' }}</small>
                </div>
              </div>
            </div>
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">بستن</button>
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
import { useAdmin } from '@/stores/modules/admin';
import { Modal } from 'bootstrap';

const store = useAdmin();
const checkPermission = store.checkPermission;

// استیت‌ها
const comments = ref({ data: [], total: 0, per_page: 10, last_page: 1 });
const stats = ref({
  total_comments: 0,
  approved: 0,
  pending: 0,
  rejected: 0,
  with_rating: 0,
  average_rating: 0,
  today_comments: 0,
  this_month: 0
});
const loading = ref(false);
const currentPage = ref(1);
const filters = ref({
  search: "",
  status: "",
  type: "",
  date_from: "",
  date_to: ""
});

// متغیرهای مودال پاسخ
const replyTarget = ref(null);
const replyContent = ref("");
const replyModal = ref(null);

// متغیرهای مودال مشاهده
const viewComment = ref(null);
const viewModal = ref(null);

// دریافت لیست کامنت‌ها
async function getComments(url) {
  loading.value = true;
  try {
    const { data } = await axios.get(url || '/comments', { 
      params: filters.value 
    });
    comments.value = data;
    currentPage.value = data.current_page || 1;
  } catch (err) {
    console.error(err);
    Swal.fire("خطا", "مشکلی در دریافت کامنت‌ها پیش آمد", "error");
  } finally {
    loading.value = false;
  }
}

// دریافت آمار
async function getStats() {
  try {
    const { data } = await axios.get('/comments-stats');
    stats.value = data;
  } catch (err) {
    console.error(err);
  }
}

// تغییر وضعیت کامنت
async function changeStatus(id, status) {
  const statusMap = {
    0: 'در انتظار',
    1: 'تایید شده',
    2: 'رد شده'
  };
  
  const result = await Swal.fire({
    title: "تغییر وضعیت",
    text: `آیا می‌خواهید وضعیت این کامنت را به "${statusMap[status]}" تغییر دهید؟`,
    icon: "question",
    showCancelButton: true,
    confirmButtonText: "بله، تغییر کن",
    cancelButtonText: "انصراف",
  });
  
  if (result.isConfirmed) {
    try {
      await axios.post(`/comments/${id}/status`, { status });
      Swal.fire("موفق", "وضعیت کامنت تغییر کرد", "success");
      getComments();
      getStats();
    } catch (err) {
      Swal.fire("خطا", "مشکلی در تغییر وضعیت پیش آمد", "error");
    }
  }
}

// نمایش مودال پاسخ
function showReplyModal(comment) {
  replyTarget.value = comment;
  replyContent.value = "";
  if (!replyModal.value) {
    const modalElement = document.getElementById('replyModal');
    replyModal.value = new Modal(modalElement);
  }
  replyModal.value.show();
}

// ارسال پاسخ
async function submitReply() {
  if (!replyContent.value.trim()) {
    Swal.fire("خطا", "لطفاً متن پاسخ را وارد کنید", "error");
    return;
  }
  
  try {
    await axios.post(`/comments/${replyTarget.value.id}/reply`, {
      content: replyContent.value
    });
    Swal.fire("موفق", "پاسخ با موفقیت ثبت شد", "success");
    replyModal.value.hide();
    getComments();
    getStats();
  } catch (err) {
    Swal.fire("خطا", "مشکلی در ارسال پاسخ پیش آمد", "error");
  }
}

// حذف کامنت
async function deleteComment(id) {
  const result = await Swal.fire({
    title: "حذف کامنت",
    text: "آیا مطمئن هستید؟ این کامنت به همراه پاسخ‌های آن حذف خواهد شد.",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  });
  
  if (result.isConfirmed) {
    try {
      await axios.post(`/comments/${id}/delete`);
      Swal.fire("موفق", "کامنت حذف شد", "success");
      getComments();
      getStats();
    } catch (err) {
      Swal.fire("خطا", err.response?.data?.message || "مشکلی در حذف پیش آمد", "error");
    }
  }
}

// تغییر صفحه
const changePage = (page) => {
  if (page) getComments(`/comments?page=${page}`);
};

// توابع کمکی
function getStatusLabel(status) {
  const map = { 0: 'در انتظار', 1: 'تایید شده', 2: 'رد شده' };
  return map[status] || 'نامشخص';
}

function getStatusBadgeClass(status) {
  const map = { 0: 'bg-warning', 1: 'bg-success', 2: 'bg-danger' };
  return map[status] || 'bg-secondary';
}

function getTypeLabel(type) {
  if (type === 'Modules\\Articles\\Models\\Article') return 'مقاله';
  if (type === 'Modules\\Products\\Models\\Product') return 'محصول';
  return 'سایر';
}

function getTypeBadgeClass(type) {
  if (type === 'Modules\\Articles\\Models\\Article') return 'bg-primary';
  if (type === 'Modules\\Products\\Models\\Product') return 'bg-success';
  return 'bg-secondary';
}

function truncateText(text, length) {
  if (!text) return '';
  return text.length > length ? text.substring(0, length) + '...' : text;
}

function formatDate(date) {
  if (!date) return '';
  const d = new Date(date);
  return d.toLocaleDateString('fa-IR');
}

function formatTime(date) {
  if (!date) return '';
  const d = new Date(date);
  return d.toLocaleTimeString('fa-IR', { hour: '2-digit', minute: '2-digit' });
}

// مقداردهی اولیه
onMounted(() => {
  getComments();
  getStats();
});
</script>

<style scoped>
.table th,
.table td {
  vertical-align: middle;
}

.table td {
  padding: 0.5rem;
}

.comment-content {
  max-width: 250px;
}

.comment-content div {
  word-wrap: break-word;
}

.badge {
  font-size: 0.8rem;
  padding: 0.3rem 0.6rem;
}

.btn-group .btn {
  padding: 0.2rem 0.5rem;
}

.modal-lg {
  max-width: 800px;
}
</style>