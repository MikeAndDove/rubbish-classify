<template>
  <div class="self-container">
    <div class="self-item"
         @click="chooseOther('otherLoc')">
      <span>{{self.loc.name || '选择服务站'}}</span>
      <span style="font-size:25rpx">选择其他垃圾投放点 ></span>
    </div>
    <div class="self-item"
         @click="chooseOther('userInfo')">
      <span>{{self.user.realName || '选择居民信息'}}</span>
      <span> ></span>
    </div>
    <div class="recycle-info">
      <div class="recycle-title"
           v-if="recycleList.length === 0">
        回收清单
      </div>
      <div class="recycle-title"
           v-else>
        <span style="width:40%">回收清单</span>
        <span style="flex:1">预估奖励</span>
        <div class="continue-add"
             @click="showMask=true">
          <span>继续添加</span>
          <span class="add"></span>
        </div>
      </div>
      <div class="add-wrap"
           v-if="recycleList.length === 0">
        <button @click="showMask=true">立即添加</button>
      </div>
      <div v-else
           class="recycle-lists">
        <div v-for="(item,i) in recycleList"
             :key="i"
             class="recycle-item">
          <span style="width:35%">{{item.detail}}({{item.quantity}}{{item.unit}})</span>
          <span style="width:20%;margin-right:200rpx">{{item.bonus}}环保金</span>
          <span class="minus"
                @click="deleteRecycle(item,i)"></span>
        </div>
      </div>
    </div>
    <button class="confirm"
            :class="{'confirmReady':recycleList.length > 0}"
            @click="confirmPush">确认投放</button>
    <DrawerScreenSelf :showMask='showMask'
                      @putRecycle='putRecycle'
                      @close='showMask=false'></DrawerScreenSelf>
  </div>

</template>
<script>
import DrawerScreenSelf from './drawerScreenSelf'
import { mapState } from 'vuex'
export default {
  components: {
    DrawerScreenSelf
  },
  computed: {
    ...mapState(['self'])
  },
  data() {
    return {
      showMask: false,
      preGreenMoney: 0,
      recycleList: [],
    }
  },
  onLoad() {

  },
  methods: {
    putRecycle(item) {
      this.showMask = false
      this.preGreenMoney = (parseFloat(this.preGreenMoney) + parseFloat(item.bonus)).toFixed(1)
      let obj = {
        "unit": item.unit,
        "detail": item.detail,
        "typeId": item.typeId,
        "quantity": item.quantity,
        "bonus": item.bonus
      }
      this.recycleList.push(obj)
    },
    deleteRecycle(item, i) {
      this.preGreenMoney -= item.bonus
      this.recycleList.splice(i, 1)
    },
    async confirmPush() {
      let pointId = this.self.loc.rpid
      let userId = this.self.user.uid
      if (!pointId || !userId) {
        let content = '请选择' + (this.self.loc.rpid ? '居民' : '投放地点')
        this.$wxUtils.showModal({ content: content, showCancel: false })
        return
      }
      if (this.recycleList.length === 0) {
        this.$wxUtils.showModal({ content: '请选择要回收的物品', showCancel: false })
        return
      }
      this.$wxUtils.showModal({ title: '温馨提示', content: '厨余垃圾不予上门回收服务可回收垃圾满5kg才可预约上门回收（除废旧大家电）回收物体不明确时请参考分类', confirmText: '确认投放' })
        .then(async res => {
          if (res === 'confirm') {
            // TODO 创建自主投放
            let list = []
            for (let index = 0; index < this.recycleList.length; index++) {
              let obj = {}
              obj.typeId = this.recycleList[index].typeId
              obj.quantity = this.recycleList[index].quantity
              list.push(obj)
            }
            // todo
            let { res } = await this.$wxUtils.request(this.$api.CreateSelfhelpList, this, { list, pointId, userId })
            wx.showToast({
              title: res.errno === 0 ? '创建成功' : res.errmsg,
              duration: 1500,
              mask: true,
            })
            this.$wxUtils.sleep(1500)
            wx.redirectTo({
              url: '/pages/home/main',
            });
          }
        })
    },
    chooseOther(type) {
      wx.navigateTo({
        url: '/pages/home/self/' + type + '/main',
      })
    }
  },
}
</script>
<style lang="less" scoped>
.self-container {
  padding-top: 50rpx;
  box-sizing: border-box;
  width: 100vw;
  height: 100vh;
  background-color: #f2f2f2;
  display: flex;
  flex-direction: column;
  align-items: center;
  .self-item {
    margin-bottom: 30rpx;
    width: 80%;
    padding: 30rpx;
    display: flex;
    justify-content: space-between;
    align-items: center;
    background-color: #fff;
  }
  .recycle-info {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    background-color: #fff;
    padding: 20rpx;
    width: 90%;
    .recycle-title {
      width: 100%;
      display: flex;
      align-items: center;
      justify-content: space-between;
      padding: 20rpx 0;
      .continue-add {
        display: flex;
        align-items: center;
        .add {
          margin-left: 20rpx;
          display: block;
          width: 30rpx;
          height: 30rpx;
          border: 2rpx solid #000;
          background-color: #000;
          border-radius: 50%;
          position: relative;
          &:before,
          &:after {
            position: absolute;
            left: 14rpx;
            top: 5rpx;
            content: " ";
            height: 25rpx;
            width: 5rpx;
            background-color: #fff;
          }
          &:before {
            transform: rotate(90deg);
          }
        }
      }
    }
    .recycle-lists {
      overflow: scroll;
      width: 100%;
      height: 300rpx;
      margin-top: 30rpx;
      margin-bottom: 50rpx;
      .recycle-item {
        width: 100%;
        display: flex;
        align-items: center;
        justify-content: space-between;
        margin: 20rpx 0;

        .minus {
          display: block;
          width: 30rpx;
          height: 30rpx;
          border: 2rpx solid #000;
          border-radius: 50%;
          position: relative;
          &:before {
            position: absolute;
            left: 15rpx;
            top: 5rpx;
            content: " ";
            height: 25rpx;
            width: 5rpx;
            background-color: #333;
            transform: rotate(90deg);
          }
        }
      }
    }
    .add-wrap {
      margin: 50rpx 0;
      width: 60%;
      button {
        border-radius: 50rpx;
        width: 80%;
        margin-bottom: 30rpx;
        border: 2rpx solid #eee;
      }
    }
    .leave-message {
      border-top: 2rpx solid #eee;
      padding-top: 20rpx;
      width: 100%;
      textarea {
        width: 100%;
        height: 100rpx;
        border: 2rpx solid #eee;
      }
    }
  }
  .confirm {
    margin-top: 30rpx;
    width: 90%;
    color: #fff;
    background-color: #d7d7d7;
  }
  .confirmReady {
    background-color: #339999;
  }
}
</style>

