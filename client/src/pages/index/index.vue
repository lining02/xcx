<template>
  <view className="index">
    <picker
      mode="date"
      fields="month"
      @change="e => onDateChange(e.detail.value, 'start')"
    >
      <AtList>
        <AtListItem title="请选择开始日期" :extraText="query.start" />
      </AtList>
    </picker>
    <picker
      mode="date"
      fields="month"
      @change="e => onDateChange(e.detail.value, 'end')"
    >
      <AtList>
        <AtListItem title="请选择结束日期" :extraText="query.end" />
      </AtList>
    </picker>
    <picker
      mode="selector"
      :range="vendorList.map(o => o.name)"
      @change="e => onDateChange(e.detail.value, 'vendor')"
    >
      <AtList>
        <AtListItem title="选择厂商" :extraText="query.vendor_name" />
      </AtList>
    </picker>
    <view v-for="item in queryList" :key="item.time" class="wrap">
      <view class="title">
        <text>{{ item.startTime }}</text>
        <text>收入 {{ item.price }}</text>
      </view>
      <view v-for="o in item.list" :key="o.time" class="inner">
        <view>
          <text>摘要字段数据</text>
          <text>+¥ {{ o.price }}</text>
        </view>
        <view>
          <text>{{ o.dec }}</text>
          <text>{{ o.data }}</text>
        </view>
      </view>
    </view>
  </view>
</template>

<script>
import "./index.less";
import { AtList, AtListItem } from "taro-ui-vue";
import "taro-ui-vue/dist/style/components/float-layout.scss";
import "taro-ui-vue/dist/style/components/list.scss";
export default {
  name: "Index",
  components: {
    AtList,
    AtListItem
  },
  onLoad: function() {
    if (!wx.cloud) {
      wx.redirectTo({
        url: "../chooseLib/chooseLib"
      });
      return;
    }
    this.initData(); // 初始化 开始结束日期
    this.getUserInfo();
    this.getVendorList();
  },
  data() {
    return {
      queryList: [],
      vendorList: [],
      query: {
        start: "",
        end: "",
        vendor_name: "全部"
      }
    };
  },
  methods: {
    onDateChange(val, key) {
      if (key === "vendor") {
        this.query.vendor_name = this.vendorList[val].name;
        this.query.vendor_id = this.vendorList[val]._id;
      } else {
        this.query[key] = val;
      }
      this.getData();
    },
    getUserInfo: function() {
      // 获取用户信息
      wx.getSetting({
        success: res => {
          if (res.authSetting["scope.userInfo"]) {
            // 已经授权，可以直接调用 getUserInfo 获取头像昵称，不会弹框
            wx.getUserInfo({
              success: res => {
                console.log({
                  avatarUrl: res.userInfo.avatarUrl,
                  userInfo: res.userInfo
                });
              }
            });
          }
        }
      });
    },
    getData() {
      let query = {};
      this.query.vendor_name !== "全部" &&
        (query.vendor_name = this.query.vendor_name);
      const db = wx.cloud.database();
      const _ = db.command;
      query.data = _.gte(this.query.start.replace(/\-/g, "/")).and(
        _.lte(this.query.end.replace(/\-/g, "/"))
      );
      db.collection("deal")
        .where(query)
        .get({
          success: res => {
            this.dealData(Array.from(res.data));
          },
          fail: err => {
            wx.showToast({
              icon: "none",
              title: "查询记录失败"
            });
            console.error("[数据库] [查询记录] 失败：", err);
          }
        });
    },
    dealData(initList) {
      let queryList = [];
      initList.map(o => {
        let idx = queryList.findIndex(
          item =>
            new Date(o.data).getTime() > item.start &&
            new Date(o.data) < item.end
        );
        if (idx === -1) {
          let year = new Date(o.data).getFullYear();
          let month = new Date(o.data).getMonth() + 1;
          queryList.push({
            list: [o],
            price: o.price,
            start: new Date(`${year}/${month}`).getTime(),
            end:
              month === 12
                ? new Date(`${year + 1}/1`).getTime()
                : new Date(`${year}/${month + 1}`).getTime(),
            startTime: `${year}年${month}月`,
            endTime:
              month === 12 ? `${year + 1}年1月` : `${year}年${month + 1}月`
          });
        } else {
          queryList[idx].list.push(o);
          queryList[idx].price += o.price;
        }
      });
      console.log(queryList);
      queryList.sort(function(a, b) {
        return a.start - b.start;
      });
      this.queryList = queryList;
    },
    initData() {
      let nowTime = new Date();
      let year = nowTime.getFullYear();
      let month = nowTime.getMonth() + 1;
      this.query.start = `${year}-${month}`;
      this.query.end = month === 12 ? `${year + 1}-01` : `${year}-${month}`;
      this.getData();
    },
    getVendorList() {
      const db = wx.cloud.database();
      db.collection("vendor")
        .where({})
        .get({
          success: res => {
            this.vendorList = Array.from(res.data);
            this.vendorList.unshift({
              name: "全部",
              _id: ""
            });
          },
          fail: err => {
            wx.showToast({
              icon: "none",
              title: "查询记录失败"
            });
            console.error("[数据库] [查询记录] 失败：", err);
          }
        });
    }
  }
};
</script>
