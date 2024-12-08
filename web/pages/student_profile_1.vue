<template>
    <v-app>
      <v-app-bar 
        color="teal" 
        height="25vh"
        class="rounded-b-xl"
        elevation="0"
      >
        <v-btn 
          icon="mdi-arrow-left" 
          color="white" 
          class="back-button"
          @click="goBack"
        ></v-btn>
        
        <div class="profile-section">
          <div class="text-center">
            <div class="text-h6 text-white mb-6">Student Profile</div>
            
            <v-avatar 
              size="80" 
              class="mb-4 white"
            >
              <v-img
                :src="selectedImage"
                alt="Profile Picture"
                cover
              ></v-img>
  
              <!-- เพิ่ม input type file แบบซ่อน -->
              <input
                type="file"
                ref="fileInput"
                accept="image/*"
                @change="onFileSelected"
                style="display: none"
              >
  
              <!-- ปุ่มแก้ไขที่จะเรียก input file -->
              <v-dialog v-model="isDialogOpen" max-width="500">
              <template v-slot:activator="{ props: activatorProps }">
                <v-btn
                  v-bind="activatorProps"
                  icon="mdi-pencil"
                  size="small"
                  class="avatar-edit-btn"
                  color="white"
                  variant="flat"
                ></v-btn>
              </template>

              <v-card title="แก้ไขรูปโปรไฟล์">
                <v-container>
                  <v-form @submit.prevent="uploadSingleFile">
                    <v-file-input
                      v-model="singleFile"
                      label="อัพโหลด"
                      accept="image/*"
                      required
                      @change="previewSingleFile"
                    ></v-file-input>

                    <!-- แสดงตัวอย่างภาพ -->
                    <!-- <v-img v-if="singleFilePreview" :src="singleFilePreview" max-width="200" max-height="200" class="my-4"></v-img> -->
                  </v-form>

                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn color="red-lighten-2" text @click="isDialogOpen = false">ปิด</v-btn>
                    <v-btn color="primary" text @click="save">บันทึก</v-btn>
                  </v-card-actions>
                </v-container>
              </v-card>
            </v-dialog>
              
            </v-avatar>
            
            <div class="white--text text-h6 mb-1">{{fullname}}</div>
            <div class="white--text text-subtitle-2">{{username}}</div>
          </div>
        </div>
      </v-app-bar>
  
      <v-main class="bg-grey-lighten-4">
        <v-container>
          <!-- General Settings Section -->
          <div class="text-subtitle-1 mb-2">General Settings</div>
          <v-card flat class="mb-4">
            <v-list>
              <v-list-item>
                <template v-slot:prepend>
                  <v-icon>mdi-theme-light-dark</v-icon>
                </template>
                <v-list-item-title>Mode</v-list-item-title>
                <v-list-item-subtitle>Dark & Light</v-list-item-subtitle>
                <template v-slot:append>
                  <v-switch color="primary"></v-switch>
                </template>
              </v-list-item>
  
              <v-list-item link>
                <template v-slot:prepend>
                  <v-icon>mdi-key</v-icon>
                </template>
                <v-list-item-title>Change Password</v-list-item-title>
                <template v-slot:append>
                  <v-icon>mdi-chevron-right</v-icon>
                </template>
              </v-list-item>
  
              <v-list-item link>
                <template v-slot:prepend>
                  <v-icon>mdi-translate</v-icon>
                </template>
                <v-list-item-title>Language</v-list-item-title>
                <template v-slot:append>
                  <v-icon>mdi-chevron-right</v-icon>
                </template>
              </v-list-item>
            </v-list>
          </v-card>
  
          <!-- Information Section -->
          <div class="text-subtitle-1 mb-2">Information</div>
          <v-card flat>
            <v-list>
              <v-list-item link>
                <template v-slot:prepend>
                  <v-icon>mdi-information</v-icon>
                </template>
                <v-list-item-title>About App</v-list-item-title>
                <template v-slot:append>
                  <v-icon>mdi-chevron-right</v-icon>
                </template>
              </v-list-item>
  
              <v-list-item link>
                <template v-slot:prepend>
                  <v-icon>mdi-file-document</v-icon>
                </template>
                <v-list-item-title>Terms & Conditions</v-list-item-title>
                <template v-slot:append>
                  <v-icon>mdi-chevron-right</v-icon>
                </template>
              </v-list-item>
  
              <v-list-item link>
                <template v-slot:prepend>
                  <v-icon>mdi-shield-check</v-icon>
                </template>
                <v-list-item-title>Privacy Policy</v-list-item-title>
                <template v-slot:append>
                  <v-icon>mdi-chevron-right</v-icon>
                </template>
              </v-list-item>
  
              <v-list-item link>
                <template v-slot:prepend>
                  <v-icon>mdi-share-variant</v-icon>
                </template>
                <v-list-item-title>Share This App</v-list-item-title>
                <template v-slot:append>
                  <v-icon>mdi-chevron-right</v-icon>
                </template>
              </v-list-item>
            </v-list>
          </v-card>
        </v-container>
      </v-main>
    </v-app>
  </template>
  
  <script setup>
  // import profileImage from '@/assets/images/profile1.png' // แก้ชื่อไฟล์ตามที่คุณมี
  import axios from 'axios'
  import { ref, onMounted } from 'vue'
  import defaultImage from '@/assets/images/profile1.png' // รูปเริ่มต้น
  import { useRouter } from 'vue-router'

  const router = useRouter()

  const isDialogOpen = ref(false)
  // const userData = ref('')
  const fullname = ref('')
  const username = ref('')
  const singleFile = ref(null)
  const singleFilePreview = ref(null)
  const selectedImage = ref('')
  // ref สำหรับ input file
  const fileInput = ref(null)
  
  const save = () => {
    uploadSingleFile()
    isDialogOpen.value = false
  }

  const uploadSingleFile = async () => {
  if (singleFile.value) {
    const formData = new FormData();
    formData.append('picture', singleFile.value);
    formData.append('username', username.value);

    try {
      const response = await axios.post('http://localhost:7000/upload-single', formData, {
        headers: {
          'Content-Type': 'multipart/form-data',
        },
      });
      console.log('Response:', response.data);
      alert(response.data.message);

      if (response.data.status === 1) {
        selectedImage.value = `http://localhost:7000/uploads/profile/${response.data.file}`;
      }
    } catch (error) {
      console.error('Upload single file error:', error);
      alert('เกิดข้อผิดพลาดในการอัปโหลด');
    }
  }
};

  const loadUsername = () => {
    const storedUsername = localStorage.getItem('username')
    username.value = storedUsername
  }

  const checkUser = async () => {
  try {
    const response = await axios.post('http://localhost:7000/checkUser', {
      username: username.value,
    });

    console.log(response.data)
    if (response.data.status === 1) {
      const user = response.data.user;
      fullname.value = user.fullname;
      username.value = user.username;
      selectedImage.value = user.picture
        ? `http://localhost:7000/uploads/profile/${user.picture}`
        : defaultImage;
    } else {
      alert(response.data.message);
    }
  } catch (error) {
    console.error('Error fetching user data:', error);
    alert('เกิดข้อผิดพลาดในการเรียกข้อมูล');
  }
};


  const previewSingleFile = (event) => {
    singleFilePreview.value = URL.createObjectURL(event)
  }

  onMounted(() => {
    loadUsername();
    checkUser();
  });

  const onFileSelected = (event) => {
  const file = event.target.files[0];
  if (file) {
    singleFile.value = file; // เก็บไฟล์ไว้ใน `singleFile`
    singleFilePreview.value = URL.createObjectURL(file); // สร้าง preview

    // หากต้องการอัปโหลดทันทีเมื่อเลือกไฟล์
    // uploadSingleFile();
  }
};

  
  // ถ้าต้องการอัพโหลดไฟล์ไปที่เซิร์ฟเวอร์
  const uploadImage = async (file) => {
    try {
      const formData = new FormData()
      formData.append('image', file)
      
      // ส่งไฟล์ไปที่เซิร์ฟเวอร์
      const response = await axios.post('http://localhost:7000/upload-single', formData)
      selectedImage.value = response.data.imageUrl
    } catch (error) {
      console.error('Error uploading image:', error)
    }
  }
  
  const goBack = () => {
    router.push('/student_list2')
  }
  </script>
  
  <style scoped>
  .back-button {
    position: absolute;
    top: 16px;
    left: 16px;
    z-index: 1;
  }
  
  .profile-section {
    width: 100%;
    padding-top: 2rem;
  }
  
  .edit-button-wrapper {
    position: relative;
    width: 80px;
    margin: -24px auto 16px;
  }
  
  .avatar-edit-btn {
    position: absolute;
    right: -5px;
    bottom: 0;
    background-color: #0158c8 !important;
    border: 2px solid white !important;
  }
  
  /* เพิ่ม style สำหรับปุ่มอัพโหลด */
  .avatar-edit-btn {
    position: absolute;
    right: -5px;
    bottom: 0;
    background-color: #0158c8 !important;
    border: 2px solid white !important;
    cursor: pointer;
    z-index: 1;
  }
  
  /* เพิ่ม hover effect */
  .avatar-edit-btn:hover {
    opacity: 0.9;
  }
  /* กำหนดให้ v-main แสดงใต้ v-app-bar */
  .v-main {
    padding-top: calc(25vh + 16px) !important;
    background-color: #f5f5f5;
  }
  
  /* กำหนดสีข้อความ */
  .white--text {
    color: white !important;
  }
  
  .text-white {
    color: white !important;
  }
  
  /* กำหนดสีพื้นหลัง avatar */
  .v-avatar.white {
    background-color: #f5f5f5 !important;
  }
  
  /* ปรับ container */
  .v-container {
    max-width: 100%;
    width: 100%;
    padding: 16px;
  }
  </style>