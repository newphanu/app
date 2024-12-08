<template>
    <v-app>
        <!-- App Bar -->
        <v-app-bar color="teal-darken-2" elevation="2">
            <template v-slot:prepend>
                <div class="rounded-circle d-flex justify-center align-center"
                    style="background-color: white; width: 40px; height: 40px;" @click="logout">
                    <v-icon color="teal-darken-2" size="24">mdi-arrow-left</v-icon>
                </div>
            </template>
            <v-app-bar-title>รายชื่อนักศึกษา</v-app-bar-title>
            <v-spacer></v-spacer>
            <!-- <v-menu>
                <template v-slot:activator="{ props }">
                    <v-btn v-bind="props" variant="text">
                        Sort by time <v-icon>mdi-chevron-down</v-icon>
                    </v-btn>
                </template>
                <v-list>
                    <v-list-item value="newest">
                        <v-list-item-title>Newest First</v-list-item-title>
                    </v-list-item>
                    <v-list-item value="oldest">
                        <v-list-item-title>Oldest First</v-list-item-title>
                    </v-list-item>
                </v-list>
            </v-menu> -->
        </v-app-bar>

        <!-- Main Content -->
        <v-main class="bg-grey-lighten-3"> <!-- เพิ่มสีพื้นหลังหลัก -->
            <v-container class="px-2">
                <div class="text-subtitle-2 text-grey-darken-1 mb-4">
                    <b style="font-size: 18px;">{{ fullname }}</b><br>
                    E1 เทคโนโลยีสารสนเทศ<br>
                    ครูที่ปรึกษา : ครูกฤษณา แนววิเศษ
                </div>

                <v-list bg-color="#f5f5f5" class="custom-list"> <!-- กำหนดสีพื้นหลังเริ่มต้น -->
                    <v-list-item v-for="(item, index) in activities" :key="index"
                        :class="{ 'border-b': index !== activities.length - 1 }" class="activity-item">
                        <template v-slot:prepend>
                            <v-avatar size="48" color="info" class="mr-3">
                                <v-img
                                    :src="`http://localhost:7000/uploads/profile/${item.picture}`"
                                ></v-img>
                            </v-avatar>
                        </template>

                        <v-list-item-title class="font-weight-medium">
                            {{ item.fullname }}
                        </v-list-item-title>
                        <v-list-item-subtitle class="text-grey-darken-1">
                            {{ item.student_id }}
                        </v-list-item-subtitle>

                        <template v-slot:append>
                            <div class="d-flex flex-column align-end">
                                <v-select
                                    v-model="item.attendance"
                                    :items="attendanceOptions"
                                    item-value="value"
                                    item-title="text"
                                    variant="text"
                                    width="90px"
                                    rounded="xl"
                                    dense
                                    hide-details
                                    :class="getAttendanceClass(item.attendance)"
                                    @change="updateAttendance(index, item.attendance)"
                                ></v-select>
                                <!-- <div class="text-grey-darken-1 text-caption mb-1">
                                    {{ item.time }}
                                    <span v-if="item.attendance !== undefined" class="ml-2">({{ getAttendanceLabel(item.attendance) }})</span>
                                </div> -->
                            </div>
                        </template>

                    </v-list-item>
                </v-list>
                <div class="d-flex justify-center">
                    <v-btn 
                        @click="submitAttendance"
                        color="primary" 
                        class="mt-4"
                        rounded="xl"
                        width="70%"
                        max-width="400"
                    >
                        บันทึกข้อมูลการเช็คชื่อ
                    </v-btn>
                </div>
            </v-container>
        </v-main>

        <!-- Bottom Navigation -->
        <v-bottom-navigation grow color="teal-darken-2">
            <v-btn value="home" @click="goList">
                <v-icon>mdi-home</v-icon>
                Home
            </v-btn>
            <v-btn value="chat">
                <v-icon>mdi-chat</v-icon>
                Chat
            </v-btn>
            <v-btn value="mail">
                <v-icon>mdi-email</v-icon>
                Mail
            </v-btn>
            <v-btn value="profile" @click="profile">
                <v-icon>mdi-account</v-icon>
                Profile
            </v-btn>
        </v-bottom-navigation>
    </v-app>
</template>

<script setup>
import axios from 'axios'
import { ref, computed , onMounted } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()
const fullname = ref('')
const attendanceOptions = [
    { text: 'ขาด', value: 3 },
    { text: 'ลา', value: 4 },
    { text: 'มา', value: 1 },
    { text: 'สาย', value: 2 }
]

// ใช้ computed property สำหรับการกำหนดคลาสของ attendance
const getAttendanceClass = (attendance) => {
    switch (attendance) {
        case 1: // "มา"
            return "attendance-green";
        case 2: // "สาย"
            return "attendance-yellow";
        case 3: // "ขาด"
            return "attendance-red";
        case 4: // "ลา"
            return "attendance-blue";
        default:
            return "";
    }
}

const activities = ref([
    {
        name: 'Claudia Alves',
        action: 'Commented on your post.',
        time: '3m ago',

    },
    {
        name: 'Claudia Alves',
        action: 'Liked your photo.',
        time: '5m ago',

    },
    {
        name: 'Claudia Alves',
        action: 'Mentioned you in a comment.',
        time: '2h ago',

    },
    {
        name: 'Dani Martinez',
        action: 'Shared your post.',
        time: '3h ago',

    },
    {
        name: 'Kimberly Nguyen',
        action: 'Commented on your photo.',
        time: '5h ago',

    },
    {
        name: 'Kimberly Nguyen',
        action: 'Started following you.',
        time: '7h ago',

    },
    {
        name: 'Mariana Napolitani',
        action: 'Saved your photo.',
        time: 'Yesterday',

    }
])

// ฟังก์ชันสำหรับการดึงข้อมูลจาก API
const listData = async () => {
    try {
        const response = await axios.get('http://localhost:7000/listStudent')
        const result = await response.data;
        // console.log('listData = ', result);
        // activities.value = result.datas
        
        activities.value = result.datas.map((item) => ({
            ...item,
            attendance: item.attendance ?? 1 // ตั้งค่าเริ่มต้นเป็น 1 หากไม่มีค่า
        }))
    } catch (error) {
        console.error('Error fetching data:', error)
    }
}

const loadFullname = () => {
    const storedFullname = localStorage.getItem('fullname')
    fullname.value = storedFullname
}

// โหลดข้อมูลเมื่อคอมโพเนนต์ถูก mount
onMounted(() => {
    loadFullname()
    listData()
})

const updateAttendance = async (index, value) => {
    activities.value[index] = {
        ...activities.value[index],
        attendance: value
    }
    // เตรียมข้อมูลที่จะส่งไปที่ API
    const payload = {
        studentName: activities.value[index].username,
        student_id: activities.value[index].student_id,
        attendance: value,
        time: activities.value[index].password
    }
    console.log('data=', payload)
}
const getAttendanceLabel = (value) => {
    const option = attendanceOptions.find((option) => option.value === value);
    return option ? option.text : "Unknown";
};


// ฟังก์ชันที่จะถูกเรียกเมื่อกดปุ่ม Submit เพื่อส่งข้อมูลทั้งหมดไปยัง API
const submitAttendance = async () => {
    // เตรียมข้อมูลที่จะส่งไปที่ API
    const payload = activities.value.map(activity => ({
        studentName: activity.username,
        student_id: activity.student_id,
        attendance: activity.attendance,
        time: activity.password
    }))

    try {
        // ส่งข้อมูลไปที่ API ด้วย axios
        const response = await axios.post("http://localhost:7000/check2", {  payload })
        console.log('API Response:', response.data)
        alert('บันทึกข้อมูลการเข้าแถวสำเร็จ')
    } catch (error) {
        console.error('Error sending data to API:', error)
    }
}

const profile = () => {
    router.push('/student_profile_1')
}

const goList = () => {
    router.push('/student_list')
} 

const logout = () => {
    localStorage.removeItem("token")
    localStorage.removeItem("fullname")
    localStorage.removeItem("username")
    router.push('/login')
}
</script>

<style scoped>
.border-b {
    border-bottom: 1px solid rgba(0, 0, 0, 0.12);
}

/* กำหนด style สำหรับ list */
.custom-list {
    border-radius: 4px;
    overflow: hidden;
}

/* กำหนด style สำหรับ list item และ hover effect */
.activity-item {
    transition: background-color 0.2s ease;
}

.activity-item:hover {
    background-color: #ffffff !important;
    cursor: pointer;
}

/* กำหนด style สำหรับ list item ตามสถานะ attendance */
.attendance-green {
    background-color: #d4edda !important; /* สีเขียว */
    border-radius: 16px; /* กำหนดขอบกลม */
}

.attendance-yellow {
    background-color: #fff3cd !important; /* สีเหลือง */
    border-radius: 16px; /* กำหนดขอบกลม */
}

.attendance-red {
    background-color: #f8d7da !important; /* สีแดง */
    border-radius: 16px; /* กำหนดขอบกลม */
}

.attendance-blue {
    background-color: #d1ecf1 !important; /* สีฟ้า */
    border-radius: 16px; /* กำหนดขอบกลม */
}

</style>