<template>
  <view class="tf-register-container">
    <AtNoticebar>学号与微信绑定后不可更改，请确认是本人学号和微信。</AtNoticebar>
    <view class="tf-register-main-part">

      <view class="tf-register-card">

        <view class="tf-register-card-title-text">注册</view>
        <view class="tf-register-card-title-tab"></view>

        <AtInput
          name="number"
          title="学号"
          type="text"
          clear
          :border="false"
          placeholder="请输入学号"
          placeholder-style="font-size: 12px;"
          class="tf-register-first-input"
          :value="studentNumber"
          :on-change="handleInputNumber"
        />

        <AtInput
          name="code"
          title="验证码"
          type="text"
          clear
          :border="false"
          :placeholder="verificationCodeHint"
          placeholder-style="font-size: 12px;"
          class="tf-register-first-input"
          :value="verificationCode"
          :on-change="handleInputCode"
        >
          <AtButton size="small" class="tf-register-code-button" :on-click="sentVerificationCode" v-if="remainedTime<=0">发送验证码</AtButton>
          <view class="tf-register-code-text" v-else>{{ remainedTime }}s后重发</view>
        </AtInput>

      </view>

      <AtButton type="primary" :on-click="onRegister">创建账户</AtButton>

    </view>

    <AtActionSheet
      :isOpened="openAuthorize"
      cancelText="取消"
      :on-cancel="handleCancel"
    >

      <AtActionSheetItem class="tf-register-action-first-item">

        <AtAvatar circle :open-data="{type: 'userAvatarUrl'}" class="tf-register-action-avatar"></AtAvatar>
        <view class="tf-register-action-nickname">
          <open-data type="userNickName"></open-data>
        </view>

      </AtActionSheetItem>

      <AtActionSheetItem>
        <AtButton type="secondary" open-type="getUserInfo" :on-get-user-info="bindGetUserInfo" style="border: none;">
          微信授权
        </AtButton>
      </AtActionSheetItem>


    </AtActionSheet>

  </view>
</template>

<script lang="ts">
import {Vue, Component} from 'vue-property-decorator';
import Taro from '@tarojs/taro';
import {APP_ROUTES} from "../../base/constant";
import {Base} from "../../base/base";
import {getLogin, postRegister, postValidateCode} from "../../base/servers/servers";

@Component({
  name: "Register"
})

export default class Register extends Vue {
  base: Base = Base.getInstance();
  studentNumber: string = '';
  verificationCode: string = '';
  openAuthorize: boolean = false;

  verificationCodeHint: string = '点击发送验证码'
  remainedTime: number = 0;

  mounted() {
    const _this = this;
    Taro.getSetting({
      success: (res: any) => {
        // 是否微信授权允许使用我的信息，不允许，true打开授权窗口
        if (!res.authSetting["scope.userInfo"]) {
          _this.openAuthorize = true;
        }
      }
    })
  }

  onRegister() {
    postRegister(this.studentNumber, this.verificationCode).then((res: any) => {
      if (res.success) {
        this.toWelcome();
      } else {
        this.base.showToast("注册失败");
      }
    }).catch((err: any) => {
      console.log(err);
    });
  }

  toWelcome() {
    Taro.navigateTo({
      url: APP_ROUTES.WELCOME
    });
  }

  bindGetUserInfo(e: any) {
    // console.log(e);
    this.openAuthorize = false;
  }

  handleCancel() {
    Taro.navigateBack();
  }

  handleInputNumber(val: string) {
    this.studentNumber = val;
  }

  handleInputCode(val: string) {
    this.verificationCode = val;
  }

  sentVerificationCode() {
    postValidateCode(this.studentNumber).then((res: any) => {
      if (res.success) {
        this.base.showToast("验证码已发送");
        this.verificationCodeHint = "已发送至学邮";
        this.remainedTime = 60;
        this.countdownTime();
      } else {
        this.base.showToast("验证码发送失败");
      }
    }).catch((err: any) => {
      console.log(err);
    });
  }

  countdownTime() {
    if (this.remainedTime <= 0) {
      this.verificationCodeHint = "点击发送验证码";
    } else {
      this.remainedTime--;
      setTimeout(() => {
        this.countdownTime();
      }, 1000);
    }
  }

}
</script>

<style lang="scss">
@import "src/assets/variables.scss";


.tf-register-container {
  height: 100%;
  background-color: $tf-color-grey1;
}

.tf-register-main-part {
  padding: 32px 48px;

  .tf-register-card {
    background-color: $tf-color-white;
    padding: 80px 24px;
    margin-bottom: 48px;

    .tf-register-card-title-text {
      color: $tf-color-primary;
      font-size: 48px
    }

    .tf-register-card-title-tab {
      background-color: $tf-color-primary;
      height: 4px;
      width: 128px;
      margin: 24px 0 200px;
    }

    .tf-register-first-input {
      margin: 48px 0;
      padding: 16px 32px;
      background-color: $tf-color-grey1;

      .tf-register-code-button {
        font-size: 24px;
        margin-left: 16px;
        border: none;
        padding: 0;
      }

      .tf-register-code-text {
        font-size: 24px;
        margin-left: 16px;
        display: flex;
        align-items: center;
        color: $tf-color-warning !important;
      }
    }
  }
}

.tf-register-action-first-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 64px 0;

  .tf-register-action-avatar {
    width: 144px;
    height: 144px;
  }

  .tf-register-action-nickname {
    margin-top: 48px;
    font-size: 32px;
  }
}


// at style
.at-input__title {
  font-size: 24px;
  width: 102px;
}

.at-input input {
  font-size: 24px;
}

.at-action-sheet__footer {
  font-size: 32px;
}

</style>
