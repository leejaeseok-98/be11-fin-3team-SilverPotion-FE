<template>
  <div class="my-page">
 

    <!-- 내 정보 섹션 -->
    <div class="profile-section section-card">
      <div class="section-header">
        <h2>내 정보</h2>
        <button class="edit-btn" @click="goToEditProfile">
          <span class="edit-icon"></span>
          수정하기
        </button>
      </div>
      
      <div class="profile-content">
        <div class="profile-image">
          <div class="image-wrapper">
            <img :src="userInfo.profileImage" alt="프로필 이미지" />
          </div>
          <button class="edit-btn2" @click="triggerFileInput">
          <span class="edit-icon"></span>
          프로필 사진 변경
        </button>
        <input type="file" ref="fileInput" @change="handleFileChange" style="display: none;">
        </div>
        
        <div class="profile-info">
          <div class="info-row">
            <div class="info-item">
              <div class="info-label">이름</div>
              <div class="info-value">{{ userInfo.name }}</div>
            </div>
            <div class="info-item">
              <div class="info-label">닉네임</div>
              <div class="info-value">{{ userInfo.nickName }}</div>
            </div>
          </div>
          
          <div class="info-row">
            <div class="info-item">
              <div class="info-label">성별</div>
              <div class="info-value">{{ userInfo.sex === 'MALE' ? '남자' : '여자' }}</div>
            </div>
            <div class="info-item">
              <div class="info-label">생년월일</div>
              <div class="info-value">{{ formatBirthday() }}</div>
            </div>
          </div>
          
          <div class="info-row">
            <div class="info-item">
              <div class="info-label">휴대폰 번호</div>
              <div class="info-value">{{ userInfo.phoneNumber }}</div>
            </div>
            <div class="info-item">
              <div class="info-label">활동 지역</div>
              <div class="info-value">{{ userInfo.region }}</div>
            </div>
          </div>
          
          <div class="info-row">
            <div class="info-item full-width">
              <div class="info-label">주소</div>
              <div class="info-value">{{ userInfo.address }} {{ userInfo.detailAddress }}</div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 보호자 섹션 -->
    <div class="connection-section section-card">
      <div class="section-header">
        <h2>연결된 보호자</h2>
        <span class="connection-count">{{ protectorList.length }}</span>
      </div>
      
      <div class="connection-list">
        <!-- 보호자 추가 버튼 -->
        <div class="connection-item add-button">
          <div class="connection-avatar add-avatar" @click="showProtectorLinkModal">
            <v-icon color="white">mdi-plus</v-icon>
          </div>
          <div class="connection-name">추가하기</div>
        </div>
        <div v-for="protector in protectorList" :key="protector.id" class="connection-item">
          <div class="connection-avatar" :style="{ backgroundColor: protector.color }" @click="showUserProfile(protector)">
            <span v-if="!protector.profileUrl">{{ protector.name.charAt(0) }}</span>
            <img v-else :src="protector.profileImg" alt="보호자 이미지" />
          </div>
          <div class="connection-name">{{ protector.name }}</div>
        </div>
      </div>
    </div>
    
    <!-- 피보호자 섹션 -->
    <div class="connection-section section-card">
      <div class="section-header">
        <h2>연결된 피보호자</h2>
        <span class="connection-count">{{ dependentList.length }}</span>
      </div>
      
      
      <div class="connection-list">
        <!-- 피보호자 추가 버튼 -->
        <div class="connection-item add-button">
          <div class="connection-avatar add-avatar" @click="showDependentLinkModal">
            <v-icon color="white">mdi-plus</v-icon>
          </div>
          <div class="connection-name">추가하기</div>
        </div>
        <div v-for="dependent in dependentList" :key="dependent.id" class="connection-item">
          <div class="connection-avatar" :style="{ backgroundColor: dependent.color }" @click="showUserProfile(dependent)">
            <span v-if="!dependent.profileImg">{{ dependent.name.charAt(0) }}</span>
            <img v-else :src="dependent.profileImg" alt="피보호자 이미지" />
          </div>
          <div class="connection-name">{{ dependent.name }}</div>
        </div>
      </div>
    </div>
    
    <!-- 힐링포션 섹션 -->
    <div class="potion-section section-card">
      <div class="section-header">
        <h2>힐링포션</h2>
      </div>
      
      <div class="potion-content">
        <div class="potion-info">
          <div class="potion-icon"></div>
          <div class="potion-details">
            <div class="potion-amount">
              <span class="amount">{{ userInfo.healingPotion }}</span>
              <span class="unit">개</span>
            </div>
            <p class="potion-desc">사용 가능한 힐링포션</p>
          </div>
        </div>
        
        <button class="charge-btn" @click="goToCharge">
          <span class="plus-icon"></span>
          충전하기
        </button>
      </div>
    </div>
  </div>

<!-- 사용자 프로필 모달 -->
<v-dialog v-model="showUserProfileModal" max-width="450">
  <v-card>
      <v-card-actions class="profile-dialog-close">
        <v-spacer></v-spacer>
        <v-btn icon @click="showUserProfileModal = false">
          <v-icon>mdi-close</v-icon>
        </v-btn>
      </v-card-actions>
      <v-card-text class="pa-0">
        <UserProfileComponent 
          :loginId="loginId" 
          :userName="userName"
          :userLongId="userLongId"
          :parentType="parentType"
          @start-text-chat="handleStartTextChat"
          @start-video-chat="handleStartVideoChat"
        />
      </v-card-text>
    </v-card>
  </v-dialog>

  <LinkRequest v-model="showLinkRequestModal" @input="handleModalChange" />
  <LinkRequestToProcteor v-model="showLinkRequestModal2" @input="handleModalChange2" />

  <!-- 스낵바 -->
  <v-snackbar v-model="snackbar.show" :color="snackbar.color" :timeout="3000">
    {{ snackbar.message }}
  </v-snackbar>

</template>

<script>
import axios from 'axios';
import UserProfileComponent from "@/components/UserProfileComponet.vue";
import LinkRequest from "@/components/LinkRequest.vue";
import LinkRequestToProcteor from "@/components/LinkRequestToProcteor.vue";

  export default {
  name: 'MyPage',
  components: {
    UserProfileComponent,
    LinkRequest,
    LinkRequestToProcteor,
  },
  data() {
    return {
      userInfo: {
        name: '',
        nickname: '',
        email: '',
        sex: '',
        phoneNumber: '',
        birthday: '',
        address: '',
        detailAddress: '',
        region: '',
        profileImage: '',
        healingPotion: 0,
      },
      protectorList: [],
      dependentList: [],
      // 사용자 프로필 모달
      showUserProfileModal: false,
      // 사용자 프로필 모달에 넘겨줄 데이터
      loginId: '',
      userName: '',
      userLongId: '',
      parentType: 'healthData',
      // 보호자/피보호자 연결 모달
      showLinkRequestModal: false,
      showLinkRequestModal2: false,
      // 스낵바
      snackbar: {
        show: false,
        message: '',
        color: 'success',
      },
    }
  },
  mounted() {
    this.bringUserInfo();
    this.bringProtector();
    this.bringDependent();
  },

  methods: {
     // 내 정보 불러오는 api
     async bringUserInfo() {
      const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/user-service/silverpotion/user/myprofile`);
      console.log(response)
      this.userInfo = response.data.result;
     },
     // 연결된 보호자 불러오는 api
     async bringProtector() {
      const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/user-service/silverpotion/user/myProtectList`);
      console.log(response)
      this.protectorList = response.data.result;
      console.log("보호자",this.protectorList.length)
    
     },
     // 연결된 피보호자 불러오는 api
     async bringDependent() {
      const response = await axios.get(`${process.env.VUE_APP_API_BASE_URL}/user-service/silverpotion/user/myDependentList`);
      console.log(response)
      this.dependentList = response.data.result;
      console.log("피보호자",this.dependentList.length)
     },
      //
      formatBirthday() {
        if (!/^\d{8}$/.test(this.userInfo.birthday)) {
          return '잘못된 형식입니다'; // 8자리 숫자가 아닌 경우
        }
        const year = this.userInfo.birthday.slice(0, 4);
        const month = this.userInfo.birthday.slice(4, 6);
        const day = this.userInfo.birthday.slice(6, 8);
        return `${year}-${month}-${day}`;
      },


//  프로필 사진 변경
     triggerFileInput() {
      this.$refs.fileInput.click();
    },
    handleFileChange(event) {
        // 사용자가 파일을 선택하면 파일 객체를 가져옴
      const file = event.target.files[0];
      if (file) {
        this.goToEditProfileImage(file);
      }
    },
    async goToEditProfileImage(file) {
        const formData = new FormData();
        formData.append('image', file);
        try{
        const response = await axios.post(`${process.env.VUE_APP_API_BASE_URL}/user-service/silverpotion/user/profileImg`, formData,{ headers: { 'Content-Type': 'multipart/form-data'  } });
        console.log(response)
        const newProfileImage = response.data.result;
        this.userInfo.profileImage = newProfileImage;
        this.showSnackbar('프로필 사진이 성공적으로 등록되었습니다.', 'success');

        }catch(error){
            console.log(error)
            this.showSnackbar('프로필 사진 등록에 실패했습니다.', 'error');
        }
    },
    
    // 보호자 연결 모달 표시
    showProtectorLinkModal() {
      this.linkRequestType = 'protector';
      this.showLinkRequestModal2 = true;
    },
    
    // 피보호자 연결 모달 표시
    showDependentLinkModal() {
      this.linkRequestType = 'dependent';
      this.showLinkRequestModal = true;
    },
    
    // 프로필 수정 페이지 이동
    goToEditProfile() {
      this.$router.push('/silverpotion/mypageupdate');
    },
    // 힐링포션 충전 페이지 이동
    goToCharge() {
      this.$router.push('/silverpotion/buyitem');
    },
    // 사용자 프로필 모달 열기
    showUserProfile(user){
      this.showUserProfileModal = true;
      console.log("사용자 프로필 모달 열기",this.showUserProfileModal)
      this.loginId = user.loginId;
      this.userName = user.name;
      this.userLongId = user.userId;
    },
     // 채팅 관련 이벤트 핸들러
     handleStartTextChat(userId) {
      console.log(`${userId}와의 1:1 채팅 시작`);
      // 채팅 페이지로 이동하거나 채팅 기능 실행
      this.showUserProfileModal = false;
    },
    
    handleStartVideoChat(userId) {
      console.log(`${userId}와의 화상 채팅 시작`);
      // 화상 채팅 기능 실행
      this.showUserProfileModal = false;
    },

    handleModalChange(val) {
      this.showLinkRequestModal = val;
    },
    handleModalChange2(val) {
      this.showLinkRequestModal2 = val;
    },

    showSnackbar(message, color) {
      this.snackbar.show = true;
      this.snackbar.message = message;
      this.snackbar.color = color;
    },

  }
}
</script>

<style scoped>
/* 전체 레이아웃 */
.my-page {
  max-width: 800px;
  margin: 0 auto;
  padding: 30px 20px;
  font-family: 'Noto Sans KR', sans-serif;
  color: #222;
  background-color: #f8f9fa;
}

.my-page-header {
  text-align: center;
  margin-bottom: 40px;
}

.my-page-header h1 {
  font-size: 2.2rem;
  margin: 0;
  font-weight: 700;
  color: #222;
  letter-spacing: -0.5px;
}


/* 공통 섹션 스타일 */
.section-card {
  background-color: white;
  border-radius: 16px;
  box-shadow: 0 3px 10px rgba(0, 0, 0, 0.05);
  padding: 24px;
  margin-bottom: 24px;
  transition: transform 0.2s, box-shadow 0.2s;
  border: 1px solid #f0f0f0;
}

.section-card:hover {
  transform: translateY(-3px);
  box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 16px;
  border-bottom: 1px solid #f0f0f0;
}

.section-header h2 {
  font-size: 1.4rem;
  font-weight: 600;
  margin: 0;
  color: #222;
  position: relative;
  padding-left: 16px;
}

.section-header h2::before {
  content: '';
  position: absolute;
  left: 0;
  top: 50%;
  transform: translateY(-50%);
  width: 4px;
  height: 18px;
  background-color: #6b46c1;
  border-radius: 2px;
}

/* 프로필 섹션 */
.profile-content {
  display: flex;
  align-items: flex-start;
}

.profile-image {
  margin-right: 30px;
  flex-shrink: 0;
}

.image-wrapper {
  width: 140px;
  height: 140px;
  border-radius: 70px;
  overflow: hidden;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
  border: 4px solid white;
  position: relative;
}

.image-wrapper::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 70px;
  box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.1);
  z-index: 1;
}

.profile-image img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.profile-info {
  flex: 1;
}

.info-row {
  display: flex;
  margin-bottom: 20px;
}

.info-row:last-child {
  margin-bottom: 0;
}

.info-item {
  flex: 1;
  min-width: 0;
  margin-right: 24px;
}

.info-item:last-child {
  margin-right: 0;
}

.info-item.full-width {
  flex-basis: 100%;
}

.info-label {
  font-size: 0.85rem;
  color: #777;
  margin-bottom: 6px;
  font-weight: 500;
}

.info-value {
  font-size: 1.1rem;
  font-weight: 600;
  color: #222;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  padding-bottom: 4px;
  border-bottom: 1px dashed #eee;
}

.edit-btn {
  background-color: #97e4f4;
  color: rgb(9, 0, 0);
  border: none;
  padding: 10px 18px;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 500;
  cursor: pointer;
  display: flex;
  align-items: center;
  transition: all 0.2s;
  box-shadow: 0 2px 6px rgba(107, 70, 193, 0.3);
}

.edit-btn:hover {
  background-color: #5a38a3;
  box-shadow: 0 4px 8px rgba(107, 70, 193, 0.4);
}

.edit-btn2 {
  background-color: #97e4f4;
  color: rgb(9, 0, 0);
  border: none;
  padding: 10px 18px;
  border-radius: 8px;
  font-size: 0.9rem;
  font-weight: 500;
  cursor: pointer;
  display: flex;
  align-items: center;
  transition: all 0.2s;
  box-shadow: 0 2px 6px rgba(107, 70, 193, 0.3);
  margin-top: 10px;
}

.edit-btn2:hover {
  background-color: #5a38a3;
  box-shadow: 0 4px 8px rgba(107, 70, 193, 0.4);
}



.edit-icon {
  display: inline-block;
  width: 16px;
  height: 16px;
  margin-right: 6px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='white' viewBox='0 0 24 24'%3E%3Cpath d='M3 17.25V21h3.75L17.81 9.94l-3.75-3.75L3 17.25zM20.71 7.04c.39-.39.39-1.02 0-1.41l-2.34-2.34c-.39-.39-1.02-.39-1.41 0l-1.83 1.83 3.75 3.75 1.83-1.83z'/%3E%3C/svg%3E");
  background-size: cover;
}

/* 연결 섹션 */
.connection-list {
  display: flex;
  flex-wrap: wrap;
  gap: 18px;
  padding: 8px 4px;
}

.connection-count {
  background-color: #f0eafb;
  color: #6b46c1;
  font-size: 0.85rem;
  font-weight: 600;
  padding: 4px 12px;
  border-radius: 30px;
}

.connection-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  width: 80px;
  transition: transform 0.2s;
}

.connection-item:hover {
  transform: translateY(-4px);
}

.connection-avatar {
  width: 70px;
  height: 70px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 10px;
  color: white;
  font-size: 1.6rem;
  font-weight: 600;
  overflow: hidden;
  box-shadow: 0 3px 8px rgba(0, 0, 0, 0.15);
  border: 3px solid white;
  position: relative;
}

.add-avatar {
  background: linear-gradient(135deg, #6b46c1 0%, #8a63d2 100%);
  cursor: pointer;
  transition: all 0.2s ease-in-out;
}

.add-avatar:hover {
  transform: scale(1.05);
  box-shadow: 0 5px 12px rgba(107, 70, 193, 0.3);
}

.add-button {
  transition: transform 0.2s;
}

.add-button:hover {
  transform: translateY(-4px);
}

.connection-avatar::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  border-radius: 50%;
  box-shadow: inset 0 0 0 1px rgba(0, 0, 0, 0.1);
}

.connection-avatar img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.connection-name {
  font-size: 0.95rem;
  text-align: center;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;
  font-weight: 500;
  color: #333;
}

/* 힐링포션 섹션 */
.potion-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background-color: #fbfaff;
  border-radius: 12px;
}

.potion-info {
  display: flex;
  align-items: center;
}

.potion-icon {
  width: 70px;
  height: 70px;
  margin-right: 22px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'%3E%3Cpath d='M40,20 v10 a20,30 0 0,0 20,30 v20 h-20 a10,10 0 0,0 0,20 h20 h20 a10,10 0 0,0 0,-20 h-20 v-20 a20,30 0 0,0 20,-30 v-10 z' fill='%23ff5c8f' /%3E%3Cpath d='M55,20 h10 v5 h-30 v-5 h10' fill='%23a78bfa' /%3E%3Cpath d='M40,30 v5 a20,20 0 0,0 20,20 v-3 a16,16 0 0,1 -16,-16 v-6 z' fill='white' opacity='0.3' /%3E%3C/svg%3E");
  background-size: contain;
  background-repeat: no-repeat;
  filter: drop-shadow(0 4px 6px rgba(255, 92, 143, 0.3));
}

.potion-details {
  display: flex;
  flex-direction: column;
}

.potion-amount {
  display: flex;
  align-items: baseline;
  margin-bottom: 8px;
}

.amount {
  font-size: 3rem;
  font-weight: 700;
  color: #222;
  line-height: 1;
}

.unit {
  font-size: 1.3rem;
  color: #555;
  margin-left: 5px;
  font-weight: 500;
}

.potion-desc {
  font-size: 0.95rem;
  color: #666;
  margin: 0;
  font-weight: 500;
}

.charge-btn {
  background-color: #ff5c8f;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 12px;
  font-size: 1rem;
  font-weight: 600;
  cursor: pointer;
  display: flex;
  align-items: center;
  transition: all 0.2s;
  box-shadow: 0 3px 8px rgba(255, 92, 143, 0.3);
}

.charge-btn:hover {
  background-color: #e54d7c;
  transform: translateY(-2px);
  box-shadow: 0 5px 12px rgba(255, 92, 143, 0.4);
}

.plus-icon {
  display: inline-block;
  width: 18px;
  height: 18px;
  margin-right: 8px;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' fill='white' viewBox='0 0 24 24'%3E%3Cpath d='M19 13h-6v6h-2v-6H5v-2h6V5h2v6h6v2z'/%3E%3C/svg%3E");
  background-size: cover;
}

/* 반응형 스타일 */
@media (max-width: 768px) {
  .profile-content {
    flex-direction: column;
    align-items: center;
  }
  
  .profile-image {
    margin-right: 0;
    margin-bottom: 24px;
  }
  
  .info-row {
    flex-direction: column;
  }
  
  .info-item {
    margin-right: 0;
    margin-bottom: 18px;
  }
  
  .potion-content {
    flex-direction: column;
    align-items: flex-start;
    padding: 16px;
  }
  
  .charge-btn {
    margin-top: 24px;
    align-self: flex-start;
    width: 100%;
    justify-content: center;
  }


}
  /* 프로필 모달 스타일 */
  .profile-dialog-close {
  position: absolute;
  top: 5px;
  right: 5px;
  z-index: 10;
}
</style>


