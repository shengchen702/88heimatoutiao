<template>
  <div>
    <el-select
    placeholder="请选择频道"
    :value="value"
    @input="$emit('input',$event)"
    >
    <!--  父组件的v-model 想实现父子组件要完成数据传输 必须定义 自定义属性value
          并绑定input事件 父组件的v-model相当于注册自定义input事件 并传$event
          实现在列表页有所有频道 在发布页没有所有频道选项 接受publish(父组件)传来的
          自定义属性 将其默认为true v-if=此属性(true) 在列表页显示 在发布页由于父组件自己将
          自定义属性includeAll的布尔值设为false 则所有频道不显示 -->
      <el-option label="所有频道" value="" v-if="includeAll"></el-option>
      <el-option
        v-for="item in channels"
        :value="item.id"
        :label="item.name"
        :key="item.id"
      >{{ item.name }}</el-option>
    </el-select>
  </div>
</template>

<script>
export default {
  props: {
    // props接收父的数据 v-model的自定义属性 value传来的数据
    // 这个数据只能用不能改 想改只能通知父组件自己修改
    value: {
      type: [String, Number],
      required: true
    },
    includeAll: {
      type: Boolean,
      default: true
    }
  },
  data () {
    return {
      channels: []
    }
  },
  methods: {
    loadChannels () {
      this.$axios({
        method: 'GET',
        url: '/channels'
      }).then(res => {
        this.channels = res.data.data.channels
      }).catch(() => {
      })
    }
  },
  created () {
    this.loadChannels()
  }
}
</script>
<style>
</style>
