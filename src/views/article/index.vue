<template>
  <div>
    <!-- 筛选部分 -->
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>筛选文章</span>
      </div>
      <el-form ref="form" label-width="80px">
        <el-form-item label="文章状态">
          <el-radio-group v-model="articleForm.status">
            <el-radio :label="null">全部</el-radio>
            <el-radio label="0">草稿</el-radio>
            <el-radio label="1">待审核</el-radio>
            <el-radio label="2">审核通过</el-radio>
            <el-radio label="3">审核失败</el-radio>
            <el-radio label="4">已删除</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="文章频道">
          <!-- <el-select placeholder="请选择频道" v-model="articleForm.channel_id">
            <el-option label="所有频道" value=""></el-option>
            <el-option v-for="item in channels" :value="item.id" :label="item.name" :key=item.id>{{ item.name }}</el-option>
          </el-select> -->
          <!-- 将组件进行封装 把频道下拉列表 封装后引入到这里 -->
          <channel-select v-model="articleForm.channel_id"></channel-select>
        </el-form-item>
        <el-form-item label="时间选择">
        <el-date-picker
          v-model="rangeDate"
          style="width:480px"
          type="daterange"
          range-separator="至"
          start-placeholder="开始时间"
          end-placeholder="结束时间"
          placeholder="选择时间范围"
          value-format="yyyy-MM-dd"
        ></el-date-picker>
        <!-- value-format设置后 v-model的输出变成一个固定格式的数组 -->
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="onQuery">查询</el-button>
        </el-form-item>
      </el-form>
    </el-card>
    <!-- 筛选部分结束 -->
    <!-- 文章列表-->
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        <span>共找到{{ totalCount }}条符合条件的内容</span>
      </div>
      <el-table :data="articles" style="width: 100%" v-loading="loading">
        <!-- 组件添加v-loading="布尔值" 实现加载效果 -->
        <el-table-column prop="date" label="封面" width="180">
          <template slot-scope="scope">
            <!-- 通过slot-scope="scope" 可以获取每一项 scope.row -->
            <img :src="scope.row.cover.images[0]" width="50" />
          </template>
        </el-table-column>
        <el-table-column prop="title" label="标题" width="180"></el-table-column>
        <el-table-column prop="status" label="状态">
          <template slot-scope="scope">
            <el-tag
              :type="articlesStatus[ scope.row.status].type"
            >{{ articlesStatus[scope.row.status].label }}</el-tag>
            <!-- 定义一个数组 按照element UI的文档格式 结合文档说明 通过scope.row获取索引
                组件内部遍历得到状态 最后实现渲染-->
          </template>
        </el-table-column>
        <el-table-column prop="pubdate" label="发布日期"></el-table-column>
        <el-table-column prop="address" label="操作">
          <template slot-scope="scope">
            <!-- 编辑文章需要传递id 在点击的同时 路由携带id跳转到publish组件 -->
            <el-button type="primary" size="mini" @click="$router.push(`/publish/${scope.row.id}`)">编辑</el-button>
            <el-button type="danger" size="mini" @click="onDelete(scope.row.id)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
      <el-pagination
      background layout="prev, pager, next"
      :total="totalCount"
      @current-change='onPageChange'
      :disabled='loading'
      :current-page="page"
      ></el-pagination>
      <!-- :disabled='布尔值' 实现加载时分页器禁用的效果 -->
    </el-card>
    <!-- 文章列表结束-->
  </div>
</template>

<script>
import channelSelect from '@/components/channel-select'
// 组件封装
export default {
  components: {
    channelSelect
  },
  data () {
    return {
      articleForm: {
        status: null,
        channel_id: '',
        begin_pubdate: '',
        end_pubdate: ''
      },
      articles: [],
      rangeDate: [],
      articlesStatus: [
        { type: 'info', label: '草稿' },
        { type: '', label: '待审核' },
        { type: 'success', label: '审核通过' },
        { type: 'warning', label: '审核失败' },
        { type: 'danger', label: '已删除' }
      ],
      totalCount: 0,
      loading: true,
      // channels: [],
      page: 1
    }
  },
  methods: {
    loadArticles (page = 1) {
      // const token = window.localStorage.getItem('user-token')
      this.loading = true
      this.$axios({
        method: 'GET',
        url: '/articles',
        // headers: {
        //   Authorization: `Bearer ${token}`
        // },
        params: {
          page,
          per_page: 10,
          status: this.articleForm.status,
          // 不选频道时需要传 null或 undefined axios就不传参 由于组件传值不能为null,
          //  应为字符串或数字 所以做判断
          channel_id: this.articleForm.channel_id ? this.articleForm.channel_id : null,
          begin_pubdate: this.rangeDate ? this.rangeDate[0] : null,
          end_pubdate: this.rangeDate ? this.rangeDate[1] : null
        }
      }).then(res => {
        this.articles = res.data.data.results
        this.totalCount = res.data.data.total_count
      }).catch(() => {
        this.$message.error('加载失败')
      }).finally(() => {
        this.loading = false
      })
    },
    // 组件的方法 可以获取当前的页数
    onPageChange (page) {
      this.page = page
      this.loadArticles(page)
    },
    // loadchannels () {
    //   this.$axios({
    //     method: 'GET',
    //     url: '/channels'
    //   }).then(res => {
    //     this.channels = res.data.data.channels
    //   }).catch(() => {
    //   })
    // },
    onDelete (id) {
      this.$confirm('此操作将永久删除该图片, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$axios({
          method: 'DELETE',
          url: `/articles/${id}`
        }).then(res => {
          this.loadArticles(this.page)
          this.$message({
            type: 'success',
            message: '操作成功!'
          })
        }).catch(() => {
          this.$message.error('操作失败!')
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    onQuery () {
      this.loadArticles(1)
      this.page = 1
    }
  },
  created () {
    this.loadArticles(1)
    // this.loadchannels()
  }
}
</script>

<style>
</style>
