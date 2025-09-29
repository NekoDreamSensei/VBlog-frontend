<template>
  <div class="public-article-list">
    <div class="header">
      <div class="search-bar">
        <el-input
          placeholder="搜索Prompt标题..."
          prefix-icon="el-icon-search"
          v-model="keywords"
          style="width: 300px"
          size="small"
          @keyup.enter.native="searchClick">
        </el-input>
        <el-button type="primary" icon="el-icon-search" size="small" style="margin-left: 10px" @click="searchClick">搜索</el-button>
      </div>
    </div>

    <div class="article-list" v-loading="loading">
      <el-table
        :data="articles"
        style="width: 100%"
        stripe
        @row-click="handleRowClick"
        v-if="articles.length > 0">
        <el-table-column prop="title" label="标题" width="400">
          <template slot-scope="scope">
            <div class="title-cell">
              <h3 class="article-title" @click.stop="viewArticle(scope.row)">{{ scope.row.title }}</h3>
              <div class="article-summary" v-html="scope.row.summary"></div>
            </div>
          </template>
        </el-table-column>
        <el-table-column prop="nickname" label="作者" width="120" align="center">
        </el-table-column>
        <el-table-column prop="cateName" label="分类" width="120" align="center">
        </el-table-column>
        <el-table-column prop="editTime" label="编辑时间" width="180" align="center">
          <template slot-scope="scope">
            {{ scope.row.editTime | formatDateTime }}
          </template>
        </el-table-column>
        <el-table-column prop="pageView" label="浏览量" width="100" align="center">
          <template slot-scope="scope">
            {{ scope.row.pageView || 0 }}
          </template>
        </el-table-column>
        <el-table-column label="标签" width="200" align="center">
          <template slot-scope="scope">
            <el-tag v-for="tag in scope.row.tags" :key="tag.id" size="mini" type="info" style="margin-right: 4px;">{{ tag.tagName }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="120" fixed="right" align="center">
          <template slot-scope="scope">
            <el-button size="mini" type="primary" @click.stop="viewArticle(scope.row)">查看</el-button>
          </template>
        </el-table-column>
      </el-table>

      <div v-if="articles.length === 0 && !loading" class="no-articles">
        <p>暂无Prompt</p>
      </div>
    </div>

    <div class="pagination" v-if="articles.length > 0">
      <el-pagination
        background
        :page-size="pageSize"
        layout="prev, pager, next"
        :total="totalCount"
        @current-change="currentChange">
      </el-pagination>
    </div>
  </div>
</template>

<script>
import { getRequest } from '../utils/api'

export default {
  name: 'PublicArticleList',
  data() {
    return {
      articles: [],
      loading: false,
      currentPage: 1,
      totalCount: 0,
      pageSize: 10,
      keywords: ''
    }
  },
  mounted() {
    this.loadArticles(1, this.pageSize)
  },
  methods: {
    loadArticles(page, count) {
      this.loading = true
      const url = `/article/public/all?page=${page}&count=${count}&keywords=${encodeURIComponent(this.keywords || '')}`

      getRequest(url).then(resp => {
        this.loading = false
        if (resp.status === 200) {
          this.articles = resp.data.articles || []
          this.totalCount = resp.data.totalCount || 0
        } else {
          this.$message.error('加载Prompt失败')
        }
      }).catch(() => {
        this.loading = false
        this.$message.error('加载Prompt失败')
      })
    },
    searchClick() {
      this.loadArticles(1, this.pageSize)
    },
    currentChange(currentPage) {
      this.currentPage = currentPage
      this.loadArticles(currentPage, this.pageSize)
    },
    viewArticle(article) {
      this.$router.push({ path: '/blogDetail', query: { aid: article.id } })
    },
    handleRowClick(row) {
      this.viewArticle(row)
    }
  }
}
</script>

<style scoped>
.public-article-list {
  padding: 20px 0;
}

.header {
  display: flex;
  justify-content: flex-end;
  align-items: center;
  margin-bottom: 20px;
}

.search-bar {
  display: flex;
  align-items: center;
}

.article-list {
  min-height: 300px;
}

/* 表格样式：类似于 Windows 资源管理器详细信息视图 */
.el-table {
  border: 1px solid #ebeef5;
  border-radius: 4px;
  overflow: hidden;
}

.el-table .el-table__header th {
  background-color: #f5f7fa;
  font-weight: bold;
  color: #303133;
  height: 40px;
}

.el-table .el-table__body-wrapper .el-table__row {
  cursor: pointer;
  transition: background-color 0.2s ease;
}

.el-table .el-table__body-wrapper .el-table__row:hover > td {
  background-color: #f5f7fa;
}

.title-cell {
  display: flex;
  flex-direction: column;
}

.article-title {
  margin: 0 0 4px 0;
  color: #303133;
  font-size: 14px;
  font-weight: 500;
  line-height: 1.4;
  cursor: pointer;
}

.article-title:hover {
  color: #20a0ff;
}

.article-summary {
  color: #909399;
  font-size: 12px;
  line-height: 1.4;
  overflow: hidden;
  display: -webkit-box;
  -webkit-line-clamp: 2;
  -webkit-box-orient: vertical;
  margin: 0;
}

.no-articles {
  text-align: center;
  padding: 40px 0;
  color: #909399;
}

.no-articles p {
  font-size: 14px;
  margin: 0;
}

.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  padding-bottom: 20px;
}

/* 响应式：小屏下列宽调整 */
@media (max-width: 768px) {
  .public-article-list {
    padding: 15px 0;
  }

  .header {
    justify-content: flex-start;
  }

  .search-bar {
    width: 100%;
  }

  .search-bar .el-input {
    flex: 1;
    margin-right: 10px;
  }

  .el-table .cell {
    font-size: 12px;
  }

  .el-table [class*="el-table__row"] td {
    padding: 8px 4px;
  }

  .article-title {
    font-size: 13px;
  }
}
</style>
