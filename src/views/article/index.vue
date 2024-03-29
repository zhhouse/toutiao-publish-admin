<template>
  <div class="article-container">
    <el-card class="filter-card">
      <div slot="header" class="clearfix">
        <el-breadcrumb separator="/">
          <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
          <el-breadcrumb-item>内容管理</el-breadcrumb-item>
        </el-breadcrumb>
      </div>
      <el-form ref="form" :model="form" label-width="80px" size="mini">
        <el-form-item label="状态">
          <el-radio-group v-model="status">
            <el-radio :label="null">全部</el-radio>
            <el-radio :label="0">草稿</el-radio>
            <el-radio :label="1">待审核</el-radio>
            <el-radio :label="2">审核通过</el-radio>
            <el-radio :label="3">审核失败</el-radio>
            <el-radio :label="4">已删除</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="频道">
          <el-select v-model="channelId" placeholder="请选择频道">
            <el-option label="全部" :value="null"></el-option>
            <el-option
              v-for="item in channels"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            ></el-option>
            <!-- <el-option label="区域二" value="beijing"></el-option> -->
          </el-select>
        </el-form-item>
        <el-form-item label="日期">
          <el-date-picker
            v-model="rangeDate"
            type="datetimerange"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            placeholder="选择日期时间"
            format="yyyy-MM-dd"
            value-format="yyyy-MM-dd"
          >
          </el-date-picker>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" :disabled="loading" @click="loadArticles(1)"
            >查询</el-button
          >
        </el-form-item>
      </el-form>
    </el-card>
    <el-card class="box-card">
      <div slot="header" class="clearfix">
        根据筛选条件共查询到{{ totalCount }}条结果：
      </div>
      <el-table
        :data="articles"
        style="width: 100%"
        class="list-table"
        size="mini"
        v-loading="loading"
      >
        <el-table-column label="封面">
          <template slot-scope="scope">
            <el-image
              style="width: 50px; height: 50px"
              :src="scope.row.cover.images[0]"
              fit="cover"
            ></el-image>
            <!-- <img v-if="scope.row.cover.images[0]" class="article-cover" :src="scope.row.cover.images[0]" alt="">
            <img v-else class="article-cover" src="./pic_bg.png" alt=""> -->
          </template>
        </el-table-column>
        <el-table-column prop="title" label="标题"></el-table-column>
        <el-table-column prop="status" label="状态">
          <template slot-scope="scope">
            <el-tag :type="articleStatus[scope.row.status].type">{{
              articleStatus[scope.row.status].text
            }}</el-tag>
            <!-- <el-tag v-else-if="scope.row.status === 1">待审核</el-tag>
            <el-tag v-else-if="scope.row.status === 2" type="success">审核通过</el-tag>
            <el-tag v-else-if="scope.row.status === 3" type="danger">审核失败</el-tag>
            <el-tag v-else-if="scope.row.status === 4" type="info">已删除</el-tag> -->
          </template>
        </el-table-column>
        <el-table-column prop="pubdate" label="发布时间"></el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button size="mini" circle icon="el-icon-edit" type="primary" />
            <el-button
              size="mini"
              circle
              icon="el-icon-delete"
              type="danger"
              @click="onRemoveArticle(scope.row.id)"
            />
          </template>
        </el-table-column>
      </el-table>

      <el-pagination
        layout="prev, pager, next"
        :total="totalCount"
        :page-size="pageSize"
        :disabled="loading"
        :current-page.sync="page"
        @current-change="onCurrentChange"
      ></el-pagination>
    </el-card>
  </div>
</template>

<script>
import { getArticles, getChannels, removeArticle } from "@/api/article";
export default {
  name: "Article",
  components: {},
  props: {},
  data() {
    return {
      articles: [],
      articleStatus: [
        { text: "草稿", type: "info" },
        { text: "待审核", type: "" },
        { text: "审核通过", type: "success" },
        { text: "审核失败", type: "wraning" },
        { text: "已删除", type: "danger" },
      ],
      totalCount: 0,
      pageSize: 20,
      status: null,
      channels: [],
      channelId: null,
      rangeDate: null,
      loading: true,
      page:1
    };
  },
  computed: {},
  watch: {},
  created() {
    this.loadArticles(1);
    this.loadChannels();
  },
  mounted() {},
  methods: {
    loadArticles(page = 1) {
      this.loading = true;
      getArticles({
        page,
        per_page: this.pageSize,
        status: this.status,
        channel_id: this.channelId,
        begin_pubdate: this.rangeDate ? this.rangeDate[0] : null,
        end_pubdate: this.rangeDate ? this.rangeDate[1] : null,
      }).then((res) => {
        const { results, total_count: totalCount } = res.data.data;
        this.articles = results;
        this.totalCount = totalCount;
        this.loading = false;
      });
    },
    loadChannels() {
      getChannels().then((res) => {
        const { channels } = res.data.data;
        this.channels = channels;
      });
    },
    onCurrentChange(page) {
      this.loadArticles(page);
    },
    onRemoveArticle(articleId) {
      this.$confirm("确认删除吗？", "删除提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          removeArticle(articleId).then((res) => {
            this.loadArticles(this.page)
            this.$message({
            type:"success",
            message:"删除成功"
          })
          })
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
  },
};
</script>

<style scoped lang="less">
.filter-card {
  margin-bottom: 30px;
}
.list-table {
  margin-bottom: 20px;
}
.article-cover {
  width: 100px;
  background-size: cover;
}
</style>
