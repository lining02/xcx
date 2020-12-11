<template>
  <view className='index'>
     <view
      v-for="item in queryList"
      :key="item.time"
      class="wrap"
    >
      <view class="title">
        <text>{{item.startTime}}</text>
        <text>收入 {{item.price}}</text>
      </view>
      <view
        v-for="o in item.list"
        :key="o.time"
        class="inner"  
      >
        <view>
          <text>摘要字段数据</text>
          <text>+¥ {{o.price}}</text>
        </view>
        <view>
          <text>{{o.dec}}</text>
          <text>{{o.data}}</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import './index.less'

export default {
  name: 'Index',
  components: {
  },
  onLoad: function() {
    if (!wx.cloud) {
      wx.redirectTo({
        url: '../chooseLib/chooseLib',
      })
      return
    }

  //  this.getUserInfo()
  //  this.getData()
  },
  data() {
    return {
      queryList:[{
        startTime: "2020年10月",
        price: 466.88,
        list: [{
          _id: "0a4429175fcda1bd00e3440a21456f31",
          _openid: "oPP4T5QMiK9lQRw28q7uVdpExGRs",
          data: "2020-10-01 01:12:00",
          dec: "储蓄卡",
          price: 233.00,
        }, {
          _id: "0a4429175fcda1bd00e3440a21456f31",
          _openid: "oPP4T5QMiK9lQRw28q7uVdpExGRs",
          data: "2020-10-02 03:12:00",
          dec: "储蓄卡",
          price: 233.88,
        }]
      }, {
        startTime: "2020年11月",
        price: 433.88,
        list: [
          {
            _id: "0a4429175fcda1bd00e3440a21456f31",
            _openid: "oPP4T5QMiK9lQRw28q7uVdpExGRs",
            data: "2020-11-01 01:12:00",
            dec: "储蓄卡",
            price: 200.88,
          }, {
            _id: "0a4429175fcda1bd00e3440a21456f31",
            _openid: "oPP4T5QMiK9lQRw28q7uVdpExGRs",
            data: "2020-11-02 03:12:00",
            dec: "储蓄卡",
            price: 233.00,
          }
        ]
      }, {
        startTime: "2020年12月",
        price: 433.88,
        list: [
          {
            _id: "0a4429175fcda1bd00e3440a21456f31",
            _openid: "oPP4T5QMiK9lQRw28q7uVdpExGRs",
            data: "2020-12-01 01:12:00",
            dec: "储蓄卡",
            price: 200.88,
          }, {
            _id: "0a4429175fcda1bd00e3440a21456f31",
            _openid: "oPP4T5QMiK9lQRw28q7uVdpExGRs",
            data: "2020-12-02 03:12:00",
            dec: "储蓄卡",
            price: 233.00,
          }
        ]
      }],
    }
  },
  methods: {
    getUserInfo: function() {
      // 获取用户信息
      wx.getSetting({
        success: res => {
          if (res.authSetting['scope.userInfo']) {
            // 已经授权，可以直接调用 getUserInfo 获取头像昵称，不会弹框
            wx.getUserInfo({
              success: res => {
                this.setData({
                  avatarUrl: res.userInfo.avatarUrl,
                  userInfo: res.userInfo
                })
              }
            })
          }
        }
      })
    },
    getData() {
      const db = wx.cloud.database()
      db.collection('deal').where({
      }).get({
        success: res => {
          this.dealData(Array.from(res.data))
        },
        fail: err => {
          wx.showToast({
            icon: 'none',
            title: '查询记录失败'
          })
          console.error('[数据库] [查询记录] 失败：', err)
        }
      })
    },
    dealData(initList) {
      let queryList = []
      initList.map(o => {
        let idx = queryList.findIndex(item => new Date(o.data).getTime() > item.start && new Date(o.data) < item.end)
        if(idx === -1) {
            let year = new Date(o.data).getFullYear()
            let month = new Date(o.data).getMonth() + 1
            queryList.push({
                list: [o],
                price: o.price,
                start: new Date(`${year}/${month}`).getTime(),
                end: month === 12 ? new Date(`${year+1}/1`).getTime() : new Date(`${year}/${month+1}`).getTime(),
                startTime: `${year}年${month}月`,
                endTime: month === 12 ? `${year+1}年1月` :`${year}年${month+1}月`
            })
        } else {
            queryList[idx].list.push(o)
            queryList[idx].price += o.price
        }
      })
      console.log(queryList)
      queryList.sort(function(a,b){
        return a.start - b.start;
      })
      this.queryList = queryList
    }
  }
}
</script>
