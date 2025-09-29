<style scoped>
.blog-table-container {
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

.blog_table_footer {
  display: flex;
  box-sizing: content-box;
  padding-top: 10px;
  padding-bottom: 0px;
  margin-bottom: 0px;
  justify-content: space-between;
}

.pagination {
  display: flex;
  justify-content: center;
  margin-top: 20px;
  padding-bottom: 20px;
}

/* 响应式：小屏下列宽调整 */
@media (max-width: 768px) {
  .blog-table-container {
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
<template>
  <div class="blog-table-container">
    <div class="header">
      <div class="search-bar">
        <el-input
          placeholder="通过标题搜索该分类下的博客..."
          prefix-icon="el-icon-search"
          v-model="keywords" 
          style="width: 300px" 
          size="small">
        </el-input>
        <el-button type="primary" icon="el-icon-search" size="small" style="margin-left: 10px" @click="searchClick">搜索</el-button>
      </div>
    </div>

    <div class="article-list" v-loading="loading">
      <el-table
        ref="multipleTable"
        :data="articles"
        tooltip-effect="dark"
        style="width: 100%"
        stripe
        @selection-change="handleSelectionChange"
        v-if="articles.length > 0">
        <el-table-column
          type="selection"
          width="35" align="left" v-if="showEdit || showDelete">
        </el-table-column>
        <el-table-column
          label="标题"
          width="400" align="left">
          <template slot-scope="scope">
            <div class="title-cell">
              <h3 class="article-title" @click="itemClick(scope.row)">{{ scope.row.title }}</h3>
              <div class="article-summary" v-html="scope.row.summary"></div>
            </div>
          </template>
        </el-table-column>
        <el-table-column
          label="编辑时间" width="180" align="center">
          <template slot-scope="scope">{{ scope.row.editTime | formatDateTime}}</template>
        </el-table-column>
        <el-table-column
          prop="nickname"
          label="作者"
          width="120" align="center">
        </el-table-column>
        <el-table-column
          prop="cateName"
          label="分类"
          width="120" align="center">
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
        <el-table-column label="操作" width="180" fixed="right" align="center" v-if="showEdit || showDelete">
          <template slot-scope="scope">
            <div style="display: flex; gap: 4px; flex-wrap: wrap; justify-content: center;">
              <el-button
                size="mini"
                @click="handleEdit(scope.$index, scope.row)" v-if="showEdit">编辑
              </el-button>
              <el-button
                size="mini"
                @click="handleRestore(scope.$index, scope.row)" v-if="showRestore">还原
              </el-button>
              <el-button
                size="mini"
                type="danger"
                @click="handleDelete(scope.$index, scope.row)" v-if="showDelete">删除
              </el-button>
            </div>
          </template>
        </el-table-column>
      </el-table>

      <div v-if="articles.length === 0 && !loading" class="no-articles">
        <p>暂无Prompt</p>
      </div>
    </div>

    <div class="blog_table_footer" v-if="articles.length > 0">
      <el-button type="danger" size="mini" style="margin: 0px;" v-show="this.articles.length>0 && showDelete"
                 :disabled="this.selItems.length==0" @click="deleteMany">批量删除
      </el-button>
      <span></span>
      <el-pagination
        background
        :page-size="pageSize"
        layout="prev, pager, next"
        :total="totalCount" @current-change="currentChange">
      </el-pagination>
    </div>
  </div>
</template>

<script>
  import {putRequest} from '../utils/api'
  import {getRequest} from '../utils/api'
//  import Vue from 'vue'
//  var bus = new Vue()

  export default{
    data() {
      return {
        articles: [],
        selItems: [],
        loading: false,
        currentPage: 1,
        totalCount: -1,
        pageSize: 6,
        keywords: '',
        dustbinData: []
      }
    },
    mounted: function () {
      var _this = this;
      this.loading = true;
      this.loadBlogs(1, this.pageSize);
      var _this = this;
      window.bus.$on('blogTableReload', function () {
        _this.loading = true;
        _this.loadBlogs(_this.currentPage, _this.pageSize);
      })
    },
    methods: {
      searchClick(){
        this.loadBlogs(1, this.pageSize);
      },
      itemClick(row){
        this.$router.push({path: '/blogDetail', query: {aid: row.id}})
      },
      deleteMany(){
        var selItems = this.selItems;
        for (var i = 0; i < selItems.length; i++) {
          this.dustbinData.push(selItems[i].id)
        }
        this.deleteToDustBin(selItems[0].state)
      },
      //翻页
      currentChange(currentPage){
        this.currentPage = currentPage;
        this.loading = true;
        this.loadBlogs(currentPage, this.pageSize);
      },
      loadBlogs(page, count){
        var _this = this;
        var url = '';
        if (this.state == -2) {
          url = "/admin/article/all" + "?page=" + page + "&count=" + count + "&keywords=" + this.keywords;
        } else {
          url = "/article/all?state=" + this.state + "&page=" + page + "&count=" + count + "&keywords=" + this.keywords;
        }
        getRequest(url).then(resp=> {
          _this.loading = false;
          if (resp.status == 200) {
            _this.articles = resp.data.articles;
            _this.totalCount = resp.data.totalCount;
          } else {
            _this.$message({type: 'error', message: '数据加载失败!'});
          }
        }, resp=> {
          _this.loading = false;
          if (resp.response.status == 403) {
            _this.$message({type: 'error', message: resp.response.data});
          } else {
            _this.$message({type: 'error', message: '数据加载失败!'});
          }
        }).catch(resp=> {
          //压根没见到服务器
          _this.loading = false;
          _this.$message({type: 'error', message: '数据加载失败!'});
        })
      },
      handleSelectionChange(val) {
        this.selItems = val;
      },
      handleEdit(index, row) {
        this.$router.push({path: '/editBlog', query: {from: this.activeName,id:row.id}});
      },
      handleDelete(index, row) {
        this.dustbinData.push(row.id);
        this.deleteToDustBin(row.state);
      },
      handleRestore(index, row) {
        let _this = this;
        this.$confirm('将该文件还原到原处，是否继续？','提示',{
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        } ).then(() => {
          _this.loading = true;
          putRequest('/article/restore', {articleId: row.id}).then(resp=> {
            if (resp.status == 200) {
              var data = resp.data;
              _this.$message({type: data.status, message: data.msg});
              if (data.status == 'success') {
                window.bus.$emit('blogTableReload')//通过选项卡都重新加载数据
              }
            } else {
              _this.$message({type: 'error', message: '还原失败!'});
            }
            _this.loading = false;
          });
        }).catch(() => {
          _this.$message({
            type: 'info',
            message: '已取消还原'
          });
        });
      },
      deleteToDustBin(state){
        var _this = this;
        this.$confirm(state != 2 ? '将该文件放入回收站，是否继续?' : '永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          _this.loading = true;
          var url = '';
          if (_this.state == -2) {
            url = "/admin/article/dustbin";
          } else {
            url = "/article/dustbin";
          }
          putRequest(url, {aids: _this.dustbinData, state: state}).then(resp=> {
            if (resp.status == 200) {
              var data = resp.data;
              _this.$message({type: data.status, message: data.msg});
              if (data.status == 'success') {
                window.bus.$emit('blogTableReload')//通过选项卡都重新加载数据
              }
            } else {
              _this.$message({type: 'error', message: '删除失败!'});
            }
            _this.loading = false;
            _this.dustbinData = []
          }, resp=> {
            _this.loading = false;
            _this.$message({type: 'error', message: '删除失败!'});
            _this.dustbinData = []
          });
        }).catch(() => {
          _this.$message({
            type: 'info',
            message: '已取消删除'
          });
          _this.dustbinData = []
        });
      }
    },
    props: ['state', 'showEdit', 'showDelete', 'activeName', 'showRestore']
  }
</script>
