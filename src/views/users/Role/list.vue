<template>
  <div class="container mt-4" v-if="checkPermission(['role_view'])">

    <div class="card mb-2">
      <div class="card-header d-flex justify-content-between align-items-center mb-3">
        <h3>
          <i class="bi bi-person-rolodex"></i>
          <span>مدیریت نقش</span>
        </h3>
        <router-link to="/users/roles/create" class="btn btn-success">
          <i class="bi bi-plus"></i>
          <span>
            افزودن نقش
          </span>
        </router-link>
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
                <th>نام نقش</th>
                <th>عملیات</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="role in roles" :key="role.id">
                <td>{{ role.id }}</td>
                <td>{{ role.name }}</td>
                <td v-if="!Number(role.is_system)">
                  <router-link :to="`/users/roles/${role.id}/edit`" class="btn btn-sm btn-warning me-2">
                    <i class="bi bi-pen"></i>
                    <span> ویرایش</span>
                  </router-link>

                  <button class="btn btn-sm btn-danger" @click="deleteRole(role.id)">
                    <i class="bi bi-trash3-fill"></i>
                    <span>حذف</span>
                  </button>
                  <button class="btn btn-sm btn-success mx-2" v-if="checkPermission(['role_permission'])"
                    @click="showModalPermission(role)">
                    <i class="bi bi-person-exclamation"></i>
                    <span>
                      دسترسی ها
                    </span>
                  </button>
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
    </div>
    <Modal v-if="permissionModal" id="permissionModal" @closeModal="() => permissionModal = false" title="سطح دسترسی">
      <b-form-group label="انتخاب دسترسی ها " label-for="permission_ids">
        <Treeselect id="permission_ids" v-if="allPermissions.length" :multiple="true" v-model="permission_ids"
          :normalizer="normalizer" :options="allPermissions" placeholder="انتخاب دسترسی ها " :clearable="true"
          :valueConsistsOf="'ALL'" />
      </b-form-group>
      <button @click="savePermissions()" class="btn btn-success">
        <i class="bi bi-save"></i>
        <span>
          ذخیره دسترسی ها
        </span>
      </button>
    </Modal>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Swal from "sweetalert2";
import Modal from "@/components/shared/modal.vue";
import Treeselect from 'vue3-treeselect'
import 'vue3-treeselect/dist/vue3-treeselect.css'
import { useAdmin } from '@/stores/modules/admin';
import { toast } from "vue3-toastify"
import "vue3-toastify/dist/index.css"
const store = useAdmin();
const checkPermission = store.checkPermission;
const roles = ref([]);
const loading = ref(false);
let currentUrl = "/roles";
let selectedRoleId = ref(null);
let permissionModal = ref(false);
let permission_ids = ref([]);
function showModalPermission(role) {
  selectedRoleId.value = role.id;
  permission_ids.value = role.permissions ? role.permissions.map(per => per.id) : [];
  permissionModal.value = true;

}
const normalizer = (node) => {
  // تبدیل کلیدها به فرمت استاندارد کامپوننت
  return {
    id: node.id,
    label: node.label,
  }
}
async function savePermissions() {
  const fd = new FormData();
  fd.append("role_id", selectedRoleId.value);
  permission_ids.value.forEach((id, index) => {
    fd.append(`ids[${index}]`, id)
  });
  let res = await axios.post('/save-permissions', fd);
  permissionModal.value = false;
  if (res.data.success) {
    toast.success("با موفقیت ثبت شد ");
    getroles()
  } else {
    toast.error('خطایی در ثبت به وجود آمده است مجددا تلاش کنید')
  }
}
const getroles = async (url = currentUrl) => {
  loading.value = true;
  try {
    const { data } = await axios.get(url);
    roles.value = data.data;
    currentUrl = url;
  } catch (err) {
    console.error(err);
  } finally {
    loading.value = false;
  }
};

let allPermissions = ref([]);
async function getAllPermissions() {
  let res = await axios.get('/all-permissions');
  allPermissions.value = res.data.data;
}
getAllPermissions()

const deleteRole = (id) => {
  Swal.fire({
    title: "حذف نقش",
    text: "آیا مطمئن هستید؟",
    icon: "warning",
    showCancelButton: true,
    confirmButtonText: "بله، حذف شود",
    cancelButtonText: "انصراف",
  }).then(async (result) => {
    if (result.isConfirmed) {
      try {
        await axios.delete(`/roles/${id}`);
        Swal.fire("موفق", "نقش حذف شد", "success");
        getroles();
      } catch (err) {
        Swal.fire("خطا", "مشکلی در حذف پیش آمد", "error");
      }
    }
  });
};

onMounted(() => {
  getroles();
});
</script>