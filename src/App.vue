<script setup>
import Sidebar from './components/shared/sideBar.vue'
import Toolbar from './components/shared/toolBar.vue'
import axios from "axios";
import { getCookie } from "./tools/methods.js";
import { useRouter, useRoute } from "vue-router"
import { useAdmin } from '@/stores/modules/admin';
const router = useRouter();
const route = useRoute();
const store = useAdmin();

if (getCookie('token')) {
  axios.defaults.headers.common.Authorization = `Bearer ${getCookie('token')}`
  store.getAdminDetail();
  store.getNotifications();


} else {
  router.push("/login")
}
axios.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response && error.response.status === 401) {
      router.push({ name: "login" }); // برو به صفحه لاگین
    }
    return Promise.reject(error);
  }
);
</script>
<template>
  <div class="d-flex w-100">
    <Sidebar v-if="route.path != '/login'" />
    <div class="flex-grow-1" id="mainContent" :class="{ 'mr-0': route.path == '/login' }">
      <Toolbar v-if="route.path != '/login'" />
      <router-view></router-view>
    </div>
  </div>
</template>

<style>
div#mainContent {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  margin-right: 250px;
}

div#mainContent.mr-0 {
  margin-right: 0;
}
</style>