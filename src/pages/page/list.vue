<template>
  <div class="app-container">
    <el-button @click="$router.push({ name: 'AddPageCategory' })">添加分类</el-button>

    <el-tabs v-model="selectedCategoryName" @tab-remove="deleteCategory" @tab-click="onTabClick">
      <el-tab-pane v-for="category in categoryList" :key="category.id" :label="category.name" :name="category.name" :closable="category.name !== '全部'" />
    </el-tabs>

    <el-table :data="pageList" highlight-current-row border>
      <el-table-column label="分类" align="center" width="200">
        <template scope="{ row }">
          <el-select v-model="row.categoryId" clearable filterable @change="categoryChange(row)" placeholder="选择分类">
            <el-option v-for="category in categoryListWithoutTotal" :key="category.id" :value="category.id" :label="category.name" />
          </el-select>
        </template>
      </el-table-column>
      <el-table-column label="page" align="center" width="120">
        <template scope="{ row }">
          <el-image v-if="row.imgUrl" :src="row.imgUrl" :preview-src-list="[row.imgUrl]" width="100px" />
        </template>
      </el-table-column>
      <el-table-column label="page名" align="center" property="name" show-overflow-tooltip />
      <el-table-column label="描述" align="center" property="description" show-overflow-tooltip />
      <el-table-column label="创建时间" align="center" width="200" show-overflow-tooltip>
        <template scope="{ row }">
          {{ row.creatorNickName + ' ' + row.createTime }}
        </template>
      </el-table-column>
      <el-table-column label="操作" width="150" align="center">
        <template scope="{ row }">
          <el-button type="primary" class="el-icon-edit" @click="updatePage(row)" />
          <el-button type="danger" class="el-icon-delete" @click="deletePage(row)" />
        </template>
      </el-table-column>
    </el-table>
    <pagination v-show="total>0" :total="total" :page.sync="queryPageListForm.pageNum" :limit.sync="queryPageListForm.pageSize" @pagination="fetchPageList" />
  </div>
</template>

<script>

import { getCategoryList, deleteCategory } from '@/api/category'
import { getPageList, deletePage, updatePage } from '@/api/page'
import Pagination from '@/components/Pagination'

export default {
  components: {
    Pagination
  },
  data() {
    return {
      selectedCategoryName: '全部',
      categoryList: [
        {
          name: '全部'
        }
      ],
      pageList: [],
      total: 0,
      queryPageListForm: {
        pageNum: 1,
        pageSize: 10,
        projectId: this.$store.state.project.id,
      }
    }
  },
  computed: {
    categoryListWithoutTotal() {
      return this.categoryList.filter(category => category.name !== '全部')
    }
  },
  created() {
    this.fetchCategoryList()
    this.fetchPageList()
  },
  methods: {
    fetchCategoryList() {
      getCategoryList({
        projectId: this.$store.state.project.id,
        type: 1 // page
      }).then(response => {
        this.categoryList = this.categoryList.concat(response.data)
      })
    },
    fetchPageList() {
      getPageList(this.queryPageListForm).then(response => {
        this.pageList = response.data.data
        this.total = response.data.total
      })
    },
    onTabClick(tab) {
      const activeCategory = this.categoryList.filter(category => category.name === tab.label)[0]
      this.queryPageListForm.categoryId = activeCategory.id
      this.queryPageListForm.pageNum = 1
      this.fetchPageList()
    },
    deleteCategory(name) {
      this.$confirm('删除' + name + '？', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        const category = this.categoryList.filter(category => category.name === name)[0]
        deleteCategory(category.id).then(response => {
          this.$notify.success(response.msg)
          // 移除tab，切换到全部，重新请求全部数据
          this.categoryList.splice(this.categoryList.indexOf(category), 1)
          this.selectedCategoryName = '全部'
          this.queryPageListForm.categoryId = undefined
          this.queryPageListForm.pageNum = 1
          this.fetchPageList()
        })
      })
    },
    deletePage(page) {
      this.$confirm('删除' + page.name, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        deletePage(page.id).then(response => {
          this.$notify.success(response.msg)
          this.fetchPageList()
        })
      })
    },
    updatePage(page) {
      this.$router.push({ name: 'UpdatePage', params: { pageId: page.id }})
    },
    categoryChange(row) {
      if (row.categoryId === '') { // 清除分类
        row.categoryId = null
      }
      updatePage(row).then(response => {
        this.fetchPageList()
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
