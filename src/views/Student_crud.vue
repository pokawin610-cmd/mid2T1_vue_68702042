<template>
  <div class="container mt-4">
    <h2 class="mb-3">รายชื่อนักศึกษา</h2>
    
    <div class="mb-3">
      <button class="btn btn-primary" @click="openAddModal">
        Add <i class="bi bi-plus-circle"></i>
      </button>
    </div>

    <table class="table table-bordered table-striped">
      <thead class="table-primary">
        <tr>
           <th>รหัสนักเรียน</th>     <!-- customer_id -->
          <th>ชื่อ</th>            <!-- firstName -->
          <th>นามสกุล</th>        <!-- lastName -->
          <th>เบอร์โทร</th>       <!-- phone -->
          <th>อีเมล</th>      <!-- username -->
          <th>แก้ไข/ลบ</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in student" :key="item.student_id">
          <td>{{ item.student_id }}</td> <!-- รหัสลูกค้า -->
          <td>{{ item.first_name }}</td>   <!-- ชื่อ -->
          <td>{{ item.last_name }}</td>    <!-- นามสกุล -->
          <td>{{ item.phone }}</td>       <!-- เบอร์โทร -->
          <td>{{ item.email }}</td>    <!-- ชื่อผู้ใช้ -->
          <td>
            <button class="btn btn-warning btn-sm" @click="openEditModal(item)">
              แก้ไข
            </button>
            |
            <button class="btn btn-danger btn-sm" @click="deleteEmployee(item.student_id)">
              ลบ
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    <div v-if="loading" class="text-center"><p>กำลังโหลดข้อมูล...</p></div>
    <div v-if="error" class="alert alert-danger">{{ error }}</div>

    <!-- ✅ Modal ใช้ทั้งเพิ่ม/แก้ไข -->
    <div class="modal fade" id="editModal" tabindex="-1">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title">{{ isEditMode ? "แก้ไขข้อมูลนักเรียน" : "เพิ่มนักเรียนใหม่" }}</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal"></button>
          </div>
          <div class="modal-body">
            <form @submit.prevent="saveCustomer">
              <div class="mb-3">
                <label class="form-label">ชื่่อ</label>
                <input v-model="editEmployee.first_name" type="text" class="form-control" required>
              </div>
              <div class="mb-3">
                <label class="form-label">นามสกุล</label>
                <input v-model="editEmployee.last_name" type="text" class="form-control" required>
              </div>
              <div class="mb-3">
                <label class="form-label">เบอร์โทร</label>
                <input v-model="editEmployee.phone" type="text" class="form-control" required>
              </div>
              <div class="mb-3">
                <label class="form-label">อีเมล</label>
                <input v-model="editEmployee.email" type="email" class="form-control" required>
              </div>
              <button type="submit" class="btn btn-success">
                {{ isEditMode ? "บันทึกการแก้ไข" : "เพิ่ม" }}
              </button>
            </form>
          </div>
        </div>
      </div>
    </div>

  </div>
</template>

<script>
import { ref, onMounted } from "vue";

export default {
  name: "EmployeeList",
  setup() {
    const student = ref([]);
    const loading = ref(true);
    const error = ref(null);
    const editEmployee = ref({});
    const isEditMode = ref(false);
    let editModal = null;

    const fetchEmployee = async () => {
      try {
        const response = await fetch("http://localhost/mid2_68702042/php_api/student_crud.php");
        const result = await response.json();

        if (result.success) {
          student.value = result.data;
        } else {
          error.value = result.message;
        }
      } catch (err) {
        error.value = err.message;
      } finally {
        loading.value = false;
      }
    };

    onMounted(() => {
      fetchEmployee();
      const modalEl = document.getElementById("editModal");
      editModal = new window.bootstrap.Modal(modalEl);
    });

    // ✅ เปิด Modal เพิ่มลูกค้าใหม่
    const openAddModal = () => {
      isEditMode.value = false;
      editEmployee.value = {
        first_name: "",
        last_name: "",
        phone: "",
        email: ""
      };
      editModal.show();
    };

    // ✅ เปิด Modal แก้ไขลูกค้า
    const openEditModal = (item) => {
      isEditMode.value = true;
      editEmployee.value = { ...item};
      editModal.show();
    };

    // ✅ ใช้ฟังก์ชันเดียวสำหรับทั้งเพิ่ม/แก้ไข
    const saveCustomer = async () => {
      const url = "http://localhost/mid2_68702042/php_api/student_crud.php";
      const method = isEditMode.value ? "PUT" : "POST";

      try {
        const response = await fetch(url, {
          method,
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(editEmployee.value)
        });

        const result = await response.json();

        if (result.success) {
          alert(result.message);
          fetchEmployee();
          editModal.hide();
        } else {
          alert(result.message);
        }
      } catch (err) {
        alert("เกิดข้อผิดพลาด: " + err.message);
      }
    };

    // ✅ ลบลูกค้า
    const deleteEmployee = async (id) => {
      if (!confirm("คุณต้องการลบข้อมูลนี้ใช่หรือไม่?")) return;
      try {
        const response = await fetch("http://localhost/mid2_68702042/php_api/student_crud.php", {
          method: "DELETE",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify({ student_id: id })
        });
        const result = await response.json();
        if (result.success) {
          student.value = student.value.filter(c => c.student_id !== id);
          alert(result.message);
        } else {
          alert(result.message);
        }
      } catch (err) {
        alert("เกิดข้อผิดพลาด: " + err.message);
      }
    };

    return {
      student,
      loading,
      error,
      editEmployee,
      isEditMode,
      openAddModal,
      openEditModal,
      saveCustomer,
      deleteEmployee
    };
  }
};
</script>
