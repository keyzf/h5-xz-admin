<template>
  <div>
    <div>
      <el-date-picker
        v-model="dateTime" 
        value-format="timestamp"
        type="daterange"
        @change="changeDate"
        range-separator="至"
        start-placeholder="开始日期"
        end-placeholder="结束日期"
      ></el-date-picker>
      <el-button
        style="margin-left:10px"
        type="primary"
        class="add myBtn"
        @click="pageIndex = 1;fetch()"
      >
        <i class="el-icon-refresh"></i> 查询
      </el-button>
      <el-button type="primary" class="add myBtn" @click="dialogFormVisible = true;">
        <i class="el-icon-circle-plus"></i> 添加平台
      </el-button>
    </div>
    <div class="card">
      <h1 class="card-header">平台列表</h1>
      <el-table
        :data="items"
        stripe
        border
        :header-cell-style="{background:'#eee'}"
        style="margin-bottom:20px"
      >
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="name" label="平台名称" width="150"></el-table-column>
        <el-table-column prop="amount" label="总金额" width="150">
          <template slot-scope="scope">
            <span>{{scope.row.amount/100}}</span> 
          </template>
        </el-table-column>
        <el-table-column prop="url" label="地址">
          <template slot-scope="scope">
            <span>{{scope.row.url}}</span>
            <el-button
              class="btn-mini"
              size="mini"
              type="success"
              @click="copyShaneUrl(scope.row.url)"
              plain
            >复制</el-button>
          </template>
        </el-table-column>
        <el-table-column fixed="right"  label="操作" align="center" width="170" >
          <template slot-scope="scope">
            <el-tooltip effect="light" content="编辑" placement="bottom">
              <el-button
                size="mini"
                type="primary"
                plain
                icon="el-icon-edit"
                @click="updateAccess(scope.row)"
              ></el-button>
            </el-tooltip>
            <el-tooltip effect="light" content="删除" placement="bottom">
              <el-button
                size="mini"
                type="danger"
                plain
                icon="el-icon-delete"
                @click="remove(scope.row)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分类管理 -->
      <el-dialog    title="平台添加" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
        <el-form
          :model="ruleForm"
          :rules="rules"
          ref="ruleForm"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="平台名称" prop="name">
            <el-input v-model="ruleForm.name"></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="submitForm('ruleForm')">立即创建</el-button>
            <el-button @click="resetForm('ruleForm')">重置</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import { VueEditor } from "vue2-editor";
export default {
  components: {
    VueEditor
  },
  data() {
    return {
      dateTime: "",
      ruleForm: {
        name: ""
      },
      rules: {
        name: [{ required: true, message: "请输入平台名称", trigger: "blur" }]
      },
      dialogFormVisible: false,

      status: 0,
      items: [],
      pageSize: 8,
      total: 0,
      pageIndex: 1,
      categorypageSize: 5000,
      categorypageIndex: 1
    };
  },
  filters: {},
  methods: {
    changeDate(val) {
      this.start_time = val ? val[0] : "";
      this.end_time = val ? val[1] : "";
      this.pageIndex = 1;
      this.fetch()
    },
    copyShaneUrl(shareLink) {
      var input = document.createElement("input"); // 直接构建input
      input.value = shareLink; // 设置内容
      document.body.appendChild(input); // 添加临时实例
      input.select(); // 选择实例内容
      document.execCommand("Copy"); // 执行复制
      document.body.removeChild(input); // 删除临时实例
      this.$message({
        type: "success",
        message: "复制成功"
      });
    },
    updateAccess(row) {
      this.ruleForm = JSON.parse(JSON.stringify(row));
      this.dialogFormVisible = true;
    },
    submitForm(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$http
            .post(
              `/content/platform/${!this.ruleForm.id ? "create" : "update"}`,
              this.ruleForm
            )
            .then(res => {
              if (res.code == 200) {
                this.dialogFormVisible = false;
                this.$message({
                  type: "success",
                  message: "操作成功"
                });
                this.fetch();
              }
            });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    resetForm(formName) {
      this.$refs[formName].resetFields();
    },
    async fetch() {
      const res = await this.$http.get("/content/platform/list", {
        params: {
          page_index: this.pageIndex,
          page_size: this.pageSize ,
          start_time:this.start_time,
          end_time:this.end_time
        }
      });
      console.log(res.data);
      if (res.code == 200) {
        this.items = res.data;
        // this.total = res.data.total_count;
      }
    },
    remove(row) {
      this.$confirm(`确定要删除该平台吗？`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      }).then(async () => {
        const res = await this.$http.post(
          `/content/platform/delete?ids=${row.id}`
        );
        if (res.code == 200) {
          this.$message({
            type: "success",
            message: "删除成功!"
          });
          this.fetch();
        } else {
          this.$message({
            type: "error",
            message: `错误：${res.msg}`
          });
        }
      });
    }
  },
  created() {
    this.fetch();
  }
};
</script>
<style>
.inputwidth100 .el-form-item__content {
  width: 100%;
}
.inputPosi {
  position: absolute;
  top: 10px;
  left: 120px;
}
.editor-el-form-item__content .el-form-item__content {
  line-height: normal;
}
.iconfont {
  font-size: 22px;
  vertical-align: middle;
  margin-left: 5px;
}
.width750 .el-dialog {
  width: 750px;
}
.btn-mini.el-button--mini {
  padding: 3px 6px;
  margin-left: 10px;
}
</style>

