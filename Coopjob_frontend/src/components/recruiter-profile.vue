<template>
  
  <div class="p-6 card">
    <fieldset :disabled="!modify_profile">

  <div class="card-image">
    <figure class="image is-4by3">
      <img src="https://bulma.io/images/placeholders/1280x960.png" alt="Placeholder image">
    </figure>
  </div>
  <div class="card-content">
    <div class="media">
      <div class="media-left">


<div class="container overlay-container">
    <figure class="image is-4by3">
      <img src="https://placekitten.com/800/600" alt="Placeholder image">
      <div class="overlay-icon">
        <button class="button is-info is-large">
          <span class="icon is-large">
            <i class="fa-light fa-camera"></i>
          </span>
        </button>
      </div>
    </figure>
  </div>
  <i class="fa-light fa-camera"></i>
  <font-awesome-icon :icon="['fas', 'camera']" />
        <figure class="image">
          <img :src="imagePath(profile_image)" alt="Profile Image" class="image-preview profile_image" v-if="profile_image && !modify_profile"/>
          <img :src="imagePath(profile_image)" alt="Profile Image" class="image-preview profile_image"  v-if="profile_image && modify_profile && !profile_image_preview"/>
          <img :src="profile_image_preview" alt="Profile Image" class="image-preview profile_image" v-if="profile_image_preview && modify_profile"/>

          <div class="file has-name" v-if="modify_profile">
          <label class="file-label">
            <input class="file-input" type="file" @change="handleProfileImageUpload" accept="image/*" />
            <span class="file-cta">
              <span class="file-label">เลือกไฟล์ </span>
            </span>
            
          </label>
          </div>

        <template v-if="$v.profile_image.$error">
          <p class="help is-danger" v-if="!$v.profile_image.required">โปรดอัปโหลดรูปภาพ</p>
        </template>

        </figure>
      </div>

      <div class="media-content">
        <label class="label">ชื่อบริษัท</label>
        <input class="input" type="text"  v-model="$v.company_name.$model" :class="{ 'is-danger': $v.company_name.$error }"/>
        <template v-if="$v.company_name.$error">
          <p class="help is-danger" v-if="!$v.company_name.required">โปรดกรอกข้อมูลในช่องนี้</p>
        </template>
        
        <label class="label">อีเมล</label>
        <input class="input" type="text" v-model="$v.email.$model" :class="{ 'is-danger': $v.email.$error }"/>
          <template v-if="$v.email.$error">
            <p class="help is-danger" v-if="!$v.email.required">โปรดกรอกข้อมูลในช่องนี้</p>
            <p class="help is-danger" v-else-if="!$v.email.email">รูปแบบอีเมลไม่ถูกต้อง</p>
          </template>
      </div>
    </div>

    <div>
      <label class="label">คำอธิบาย</label>
      <textarea class="textarea" v-model="$v.description.$model" :class="{ 'is-danger': $v.description.$error }"></textarea>
      <template v-if="$v.description.$error">
        <p class="help is-danger" v-if="!$v.description.required">โปรดกรอกข้อมูลในช่องนี้</p>
      </template>
    </div>

    <label class="label">ประเภทธุกิจ</label>
    <multiselect v-model="selectedBusinessType"  placeholder="Search or add a tag" label="title" track-by="title" :options="options" :multiple="true" :taggable="true" @tag="addTag"></multiselect>

    <label class="label">วิดีโอของบริษัท</label>
      <div class="field" v-show="modify_profile">
        <div class="columns">
          <div class="column">
            <img src="https://sv1.picz.in.th/images/2023/04/02/m1nC4y.png" />
          </div>
          <div class="column">
            <img src="https://sv1.picz.in.th/images/2023/04/02/m1nvp8.png"/>
          </div>
          <div class="column">
            <img src="https://sv1.picz.in.th/images/2023/04/02/m1nJNR.png"/>
          </div>
        </div>
        <div class="control">
          <input class="input" type="text" v-model="$v.company_video.$model"/>
        </div>
      </div>
      <div
        class="field is-horizontals"
        v-show="!modify_profile"
        v-html="company_video">
      </div>

  </div>
  
    </fieldset>

    <div class="mt-6 modify_profile">
      <button
        v-show="!modify_profile"
        class="button is-medium ml-2 is-info"
        @click="modify_profile = !modify_profile"
      >
        แก้ไข
      </button>
      <button
        v-show="modify_profile"
        class="button is-medium ml-2 is-success"
        @click="saveProfile"
        :disabled="$v.$error"
      >
        บันทึก
      </button>
      <button
        v-show="modify_profile"
        class="button is-medium ml-2 is-danger"
        @click="resetProfile"
      >
        ยกเลิก
      </button>
    </div>
  </div>
</template>

<script>
import { required, email } from 'vuelidate/lib/validators';
import axios from '@/plugins/axios';
import Swal from 'sweetalert2';
import Multiselect from 'vue-multiselect'



export default {
  components: {
    Multiselect,
  },
  data() {
    return {
      company_name: '',
      email: '',
      description: '',
      profile_image: '',
      profile_image_name: '',
      profile_image_preview: '',
      company_video: '',
      modify_profile: false,
      selectedBusinessType: [],
      options: [
        { title: 'โฆษณา / ประชาสัมพันธ์'},{ title: 'เกษตรกรรม / ป่าไม้'},
        { title: 'สายการบิน'},{ title: 'ความงาม / เครื่องสำอางค์'},{ title: 'เครื่องดื่ม / อาหาร / ภัตตาคาร'},
        { title: 'อุตสาหกรรมการท่องเที่ยว'},{ title: 'องค์กรการกุศล'},{ title: 'เคมีภัณฑ์ / ปิโตรเคมีภัณฑ์'},
        { title: 'เสื้อผ้า / รองเท้า / เครื่องตกแต่ง'},{ title: 'การก่อสร้าง / การตกแต่ง'},{ title: 'สินค้าอุปโภค / บริโภค'},
        { title: 'บริการพาณิชย์อิเล็กทรอนิกส์'},{ title: 'การศึกษา'},{ title: 'อิเล็คโทรนิกส์ / เครื่องใช้ไฟฟ้า'},
        { title: 'เครื่องจักร'},{ title: 'การเงิน / การธนาคาร / หลักทรัพย์'},{ title: 'เฟอร์นิเจอร์'},
        { title: 'หน่วยงานรัฐบาล'},{ title: 'กราฟฟิก ดีไซน์'},{ title: 'โรงพยาบาล'},{ title: 'โรงแรม / รีสอร์ท'},
        { title: 'ทรัพยากรบุคคล'},{ title: 'ไอที - ฮาร์ดแวร์'},{ title: 'ไอที - ซอฟต์แวร์ / โทรคมนาคม'},
        { title: 'อัญมณี / เครื่องประดับ'},{ title: 'กฎหมาย / ที่ปรึกษาทางธุรกิจ'},{ title: 'การผลิต'},
        { title: 'การตลาด'},{ title: 'การท่องเที่ยว'},{ title: 'กีฬา / นันทนาการ'},{ title: 'อื่นๆ'},
      ]
    };
  },
  mounted() {
    this.getUserProfile();

    
    // เรียกใช้งาน Selectize.js เมื่อ Vue component ถูก mount เสร็จ
    
  },
  methods: {
    getUserProfile() {
      const token = localStorage.getItem('token');
      const config = {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      };
      axios.get('http://localhost:3000/recruiter/getData', config).then((res) => {
        const user = res.data;
        if (user[0].profile_image) {
          this.profile_image = user[0].profile_image.replace(/\\/g, '/').replace('static', '');
        }

        this.company_name = user[0].company_name;
        this.description = user[0].description;
        this.email = user[0].email;
        this.role = user[0].role;
        this.company_video = user[0].company_video;
      });
    },
    imagePath(previewProfileImage) {
      if (previewProfileImage) {
        return 'http://localhost:3000' + previewProfileImage;
      } else {
        return 'https://bulma.io/images/placeholders/640x360.png';
      }
    },

    saveProfile() {
      const token = localStorage.getItem('token');
      const config = {
        headers: {
          Authorization: `Bearer ${token}`,
        },
      };
      const formData = new FormData();
      formData.append('company_name', this.company_name);
      formData.append('email', this.email);
      formData.append('description', this.description);
      if (this.profile_image) {
        formData.append('profile_image', this.profile_image);
      }
      formData.append('company_video', this.company_video);

      axios
        .post('http://localhost:3000/recruiter/editProfile', formData, config)
        .then((response) => {
          console.log(response);

          Swal.fire({
            position: 'center',
            icon: 'success',
            title: 'บันทึกข้อมูลสำเร็จ',
            showConfirmButton: false,
          });
          this.getUserProfile();
          this.modify_profile = false;
        })
        .catch((error) => {
          console.log(error);
          Swal.fire({
            position: 'center',
            icon: 'error',
            title: 'เกิดข้อผิดพลาดในการบันทึกข้อมูล',
            showConfirmButton: false,
          });
        });
    },
    handleProfileImageUpload(event) {
      const file = event.target.files[0];
      this.profile_image = file;
      this.profile_image_name = file.name;

      // แสดงตัวอย่างรูปภาพที่อัปโหลด
      const reader = new FileReader();
      reader.onload = () => {
        this.profile_image_preview = reader.result;
      };
      reader.readAsDataURL(file);
    },
    resetProfile() {
      this.getUserProfile();
      this.modify_profile = false;
    },
    removeItem(item) {
      // ลบรายการที่ถูกเลือก
      this.selectedValues = this.selectedValues.filter(value => value !== item);
    },
    customLabel(option) {
      // กำหนด custom label สำหรับแสดงผลใน dropdown
      return `${option.name} (${option.id})`;
    },
  },
  validations: {
    company_name: {
      required,
    },
    email: {
      required,
      email,
    },
    description: {
      required,
    },
    profile_image: {
      required,
    },
    company_video: {},
  },
};
</script>
<style src="vue-multiselect/dist/vue-multiselect.min.css"></style>
<style scoped>
/* สไตล์ CSS สำหรับฟอร์ม */
.card {
  width: 80%; /* เพิ่มความกว้างของการ์ด ค่าที่เพิ่มขึ้นนี้ขึ้นอยู่กับความต้องการของคุณ */
  margin: 0 auto; /* ใช้ในการจัดวางการ์ดให้อยู่ตรงกลางหน้าจอ */
}
.profile_image {
  width: 150px;
  height: 150px;
  border: 2px solid gray;
  border-radius: 25px;
}
.field.is-horizontals {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.selected-items {
  margin-top: 10px;
}

.tag {
  margin-right: 5px;
  margin-bottom: 5px;
}

.overlay-container {
      position: relative;
    }

    .overlay-icon {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
    }
</style>