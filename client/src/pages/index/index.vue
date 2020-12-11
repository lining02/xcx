<template>
  <view className='index'>
    <Login />
  </view>
</template>

<script>
import './index.less'
import Login from '../../components/login/index.weapp'

export default {
  name: 'Index',
  components: {
    Login
  }
}
</script>
