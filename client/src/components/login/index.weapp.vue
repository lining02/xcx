<template>
    <view className='index'>
        <!-- <AtButton @tap="getLogin">获取登录云函数</AtButton> -->
        <text>context：{{context}}</text>
    </view>
</template>

<script>
import Taro from '@tarojs/taro'
// import { AtButton } from "taro-ui-vue3"

export default {
    name: 'login',
    components: {
        // AtButton
    },
    data(){
        return {
            context: {}
        }
    },
    methods: {
        getLogin () {
            Taro.cloud
                .callFunction({
                    name: "login",
                    data: {}
                })
                .then(res => {
                    this.context = JSON.stringify(res.result)
                })
        }
    },
}
</script>
