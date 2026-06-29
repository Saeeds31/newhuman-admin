<template>
  <b-navbar id="mainNavbar" variant="light" class="bg-white border-bottom px-3">
    <b-navbar-brand href="#">Admin Panel</b-navbar-brand>

    <b-navbar-nav class=" d-flex align-items-center gap-2">
      <!-- دکمه‌های اضافی -->

      <b-button variant="info" @click="router.go(-1)">
        <i class="bi-arrow-left"></i>
      </b-button>

      <b-button variant="outline-primary" size="sm" pill class="position-relative" @click="showNotificationModal">
        <i class="bi bi-bell-fill"></i>
        <b-badge v-if="unseenCount > 0" class=" bg-primary position-absolute top-0 start-100 translate-middle"
          style="font-size: 0.65rem; width:24px">
          {{ unseenCount > 99 ? '99+' : unseenCount }}
        </b-badge>

      </b-button> <!-- Logout -->
      <b-button variant="danger" @click="logout">
        <i class="bi-box-arrow-left"></i>
      </b-button>
    </b-navbar-nav>
  </b-navbar>

  <Modal v-if="modalShow" id="detailModal" @closeModal="() => modalShow = false" title="مشاهده پیام">
    <b-tabs content-class="mt-3" justified pills>
      <!-- تب اعلان‌های جدید -->
      <b-tab title="جدید" active>
        <template #title>
          <span>جدید</span>
          <b-badge v-if="unseenCount > 0" pill bg="danger" class="me-1" style="font-size: 0.65rem;">
            {{ unseenCount }}
          </b-badge>
        </template>

        <div class="notification-list" style="max-height: 60vh; overflow-y: auto;">
          <div v-if="unseenNotifications.length === 0" class="text-center py-5 text-muted">
            <i class="bi bi-bell-slash fs-1 d-block mb-3"></i>
            <p>اعلان جدیدی وجود ندارد</p>
          </div>

          <div v-for="notif in unseenNotifications" :key="notif.id"
            class="notif-item p-3 border-bottom bg-light bg-opacity-75">
            <div class=" d-flex">
              <div class="flex-shrink-0">
                <div
                  class="avatar-sm bg-primary bg-opacity-10 rounded-circle d-flex align-items-center justify-content-center">
                  <i class="bi bi-bell-fill text-primary"></i>
                </div>
              </div>
              <div class="flex-grow-1 ms-3">
                <h6 class="mb-1 fw-bold text-dark">{{ notif.title }}</h6>
                <p class="mb-1 small text-muted">{{ notif.message }}</p>
                <small class="text-primary fw-medium">
                  {{ formatJalaliTimeAgo(notif.created_at) }}
                </small>
              </div>
              <div class="align-self-center">
                <button class="badge bg-danger rounded-pill" @click="markAsSeen(notif)">
                  <span>دیده شد</span>
                  <i class="bi-check"></i>
                </button>
              </div>
            </div>
          </div>
        </div>
      </b-tab>

    </b-tabs>

    <!-- دکمه همه رو خونده شده کن -->
    <div class="p-3 border-top bg-light" v-if="unseenCount > 0">
      <b-button size="sm" variant="outline-primary" block @click="markAllAsSeen">
        <i class="bi bi-check2-all me-1"></i>
        علامت‌گذاری همه به عنوان خوانده شده
      </b-button>
    </div>
  </Modal>
</template>

<script setup>
import { BNavbar, BNavbarBrand, BNavbarNav, BButton } from 'bootstrap-vue-3'
import { useRouter } from "vue-router"
import { ref, computed } from 'vue'
import { deleteCookie } from '../../tools/methods'
import axios from 'axios'
import { toast } from 'vue3-toastify'
import { useAdmin } from '@/stores/modules/admin'
import Modal from '@/components/shared/modal.vue'

const router = useRouter()
const store = useAdmin()

const modalShow = ref(false)

// داده‌های اعلان
const notifications = computed(() => store.notifications || [])

// جدا کردن دیده نشده و دیده شده
const unseenNotifications = computed(() =>
  notifications.value.filter(n => !n.seen)
)
const seenNotifications = computed(() =>
  notifications.value.filter(n => n.seen)
)

const unseenCount = computed(() => unseenNotifications.value.length)

// باز کردن مودال
const showNotificationModal = () => {
  modalShow.value = true
}

// یکی رو خونده شده کن
const markAsSeen = (notif) => {
  if (!notif.seen) {
    notif.seen = true
    toast.success('اعلان خوانده شد')
    axios.get(`notifications/${notif.id}/as-read`);
  }
}

// همه رو خونده شده کن
const markAllAsSeen = () => {
  const fd = new FormData();
  notifications.value.forEach(n => {
    if (!n.seen) {
      fd.append('ids[]', n.id)
      n.seen = true
    }
  })
  toast.success('همه اعلان‌ها خوانده شدند')
  axios.post('notifications/mark-many-read', fd);
}

// تبدیل تاریخ به حالت "چند دقیقه پیش"
const formatJalaliTimeAgo = (dateString) => {
  const now = new Date()
  const created = new Date(dateString)
  const diffMs = now - created
  const diffMin = Math.floor(diffMs / 60000)
  const diffHour = Math.floor(diffMs / 3600000)
  const diffDay = Math.floor(diffMs / 86400000)

  if (diffMin < 1) return 'همین الان'
  if (diffMin < 60) return `${diffMin} دقیقه پیش`
  if (diffHour < 24) return `${diffHour} ساعت پیش`
  if (diffDay < 7) return `${diffDay} روز پیش`
  return created.toLocaleDateString('fa-IR')
}
const logout = () => {
  toast.success("با موفقیت خارج شدید")
  deleteCookie("token")
  delete axios.defaults.headers.common.Authorization
  router.push('/login')
}


</script>