<template>
  <div class="app-container">
    <!-- 页面标题 -->
    <div class="page-header">
      <div class="header-icon">
        <i class="el-icon-star-on"></i>
      </div>
      <div class="header-text">
        <h3>我的收藏</h3>
        <p>管理您收藏的院校和专业信息</p>
      </div>
    </div>

    <!-- 收藏列表表格 -->
    <el-table :data="storeupList" v-loading="loading" border style="width: 100%">
      <el-table-column label="收藏ID" prop="storeupId" align="center"></el-table-column>
      <el-table-column label="院校名称ID" prop="universityName" align="center"></el-table-column>
      <el-table-column label="专业名称" prop="majorName" align="center"></el-table-column>
      <el-table-column label="报名科目" prop="subject" align="center">
        <template #default="scope">
          <span>{{ scope.row.subject === 1 ? '历史类' : '物理类' }}</span>
        </template>
      </el-table-column>
      <el-table-column label="2024投档线" prop="minScore2024" align="center"></el-table-column>
      <el-table-column label="收藏时间" prop="actionTime" align="center"></el-table-column>
      <el-table-column label="操作" align="center" width="200px">
        <template #default="scope">
          <el-button type="info" size="mini" @click="handleView(scope.row)">查看详情</el-button>
          <el-button type="danger" size="mini" @click="handleDelete(scope.row.storeupId)">取消收藏</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页组件 -->
    <pagination
      v-show="total > 0"
      :total="total"
      :page.sync="queryParams.pageNum"
      :limit.sync="queryParams.pageSize"
      @pagination="fetchStoreups"
    />

    <!-- 查看详情对话框 -->
    <el-dialog :visible.sync="viewDialogVisible" title="收藏详情" width="40%">
      <el-form :model="viewStoreupForm" label-width="100px">
        <el-form-item label="院校名称">
          <el-input v-model="viewStoreupForm.universityName" readonly></el-input>
        </el-form-item>
        <el-form-item label="专业名称">
          <el-input v-model="viewStoreupForm.majorName" readonly></el-input>
        </el-form-item>
        <el-form-item label="报名科目">
          <el-input :value="viewStoreupForm.subject === 1 ? '历史类' : '物理类'" readonly></el-input>
        </el-form-item>
        <el-form-item label="2024最低投档线">
          <el-input v-model="viewStoreupForm.minScore2024" readonly></el-input>
        </el-form-item>
        <el-form-item label="专业简介">
          <el-input v-model="viewStoreupForm.description" readonly></el-input>
        </el-form-item>
        <el-form-item label="收藏时间">
          <el-input v-model="viewStoreupForm.actionTime" readonly></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="viewDialogVisible = false">关闭</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import {listStoreups, deleteStoreup} from "@/api/university/storeup";

export default {
  data() {
    return {
      loading: false,
      storeupList: [],
      total: 0,
      queryParams: {
        pageNum: 1,
        pageSize: 10,
      },
      viewDialogVisible: false,
      viewStoreupForm: {},
    };
  },
  created() {
    this.fetchStoreups();
  },
  methods: {
    // 获取收藏列表
    fetchStoreups() {
      this.loading = true;
      listStoreups(this.queryParams).then((response) => {
        this.storeupList = response.rows;
        this.total = response.total;
        this.loading = false;
      });
    },
    // 查看收藏详情
    handleView(row) {
      this.viewStoreupForm = {...row};
      this.viewDialogVisible = true;
    },
    // 取消收藏
    handleDelete(storeupId) {
      this.$confirm("确认取消收藏吗？", "提示", {
        type: "warning",
      }).then(() => {
        deleteStoreup(storeupId).then(() => {
          this.$message.success("取消收藏成功");
          this.fetchStoreups();
        });
      });
    },
  },
};
</script>

<style scoped lang="scss">
.app-container {
  padding: 20px;
}

/* 页面标题样式 */
.page-header {
  display: flex;
  align-items: center;
  padding: 20px;
  background: #fff;
  border: 1px solid #e4e7ed;
  margin-bottom: 20px;
  
  .header-icon {
    width: 48px;
    height: 48px;
    background: #1a5fb4;
    display: flex;
    align-items: center;
    justify-content: center;
    margin-right: 16px;
    
    i {
      font-size: 24px;
      color: #fff;
    }
  }
  
  .header-text {
    h3 {
      margin: 0 0 6px 0;
      font-size: 18px;
      color: #303133;
      font-weight: 600;
    }
    
    p {
      margin: 0;
      font-size: 13px;
      color: #909399;
    }
  }
}

/* 表格样式优化 */
::v-deep .el-table {
  border: 1px solid #e4e7ed;
  
  th {
    background: #f5f7fa;
    color: #303133;
    font-weight: 600;
  }
}

::v-deep .el-button {
  border-radius: 0;
}

::v-deep .el-dialog {
  border-radius: 0;
}
</style>
