<template>
  <div id="table-config-list">
    <!--上部查询-->
    <el-col :span="24">
      <el-form :inline="true" :model="tableTopForm" class="demo-form-inline" size="medium">
        <el-form-item>
          <el-input v-model="tableTopForm.inputKey" placeholder="同步模糊搜索" clearable></el-input>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="getTableConfigListBtn">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="TableConfigAddBtn">新增</el-button>
        </el-form-item>
      </el-form>
    </el-col>
    <!--中部列表-->
    <el-table :data="table" v-loading="tableLoading" border stripe size="small"
              style="width: 100%;">
      <el-table-column prop="id" v-if="false">
      </el-table-column>
      <el-table-column prop="name" label="任务名称">
      </el-table-column>
      <el-table-column prop="tableto" label="目标表">
      </el-table-column>
      <el-table-column
        label="上次同步时间"
        width="180">
        <template slot-scope="scope">
          <i class="el-icon-time"></i>
          <span style="margin-left: 10px">{{ scope.row.lasttime }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="上次同步耗时"
        width="100">
        <template slot-scope="scope">
          <el-popover trigger="hover" placement="top">
            <p>查询耗时: {{ scope.row.sqlquerytime }}</p>
            <p>计算耗时: {{ scope.row.computetime }}</p>
            <p>更新耗时: {{ scope.row.sqlruntime }}</p>
            <div slot="reference" class="name-wrapper" >
              <el-tag size="small" style="width: 75px">{{ scope.row.timecost }}</el-tag>
            </div>
          </el-popover>
        </template>
      </el-table-column>
      <!--<el-table-column prop="synflag" label="同步开关" width="80">-->
        <!--<template slot-scope="scope">-->
          <!--<el-switch-->
            <!--v-model="scope.row.synflag">-->
          <!--</el-switch>-->
        <!--</template>-->
      <!--</el-table-column>-->
      <el-table-column label="操作" width="250px">
        <template slot-scope="scope">
          <el-button size="mini" @click="testTableConfigBtn(scope.$index, scope.row)">测试</el-button>
          <el-button size="mini" @click="editTableConfigBtn(scope.$index, scope.row)">编辑</el-button>
          <el-button size="mini" type="danger" @click="deleteTableConfigBtn(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
      <el-table-column prop="synmsg" label="同步信息">
      </el-table-column>
    </el-table>

    <table-config-save ref="dialog" v-on:reloadTable="getTableConfigList"></table-config-save>

  </div>
</template>

<script>
  import TableConfigSave from './table-config-save.vue'
  import {deleteTableConfig, getDataSourceList, getTableList, testTableSyn} from "../../api/api";
  export default {
    components: {
      TableConfigSave
    },
    data() {
      return {
        /**上部菜单栏*/
        tableTopForm: {
          inputKey: '',
        },
        /**列表相关*/
        table: [],
        tableLoading: false,
      }
    },
    methods: {
      /**
       * 获取同步列表函数
       */
      getTableConfigList: function () {
        let para = {
          inputKey: this.inputKey,
        };
        this.tableLoading = true;
        getTableList(para).then((res) => {
          this.tableLoading = false;
          if(res.code === '0'){
            this.jsonToArray(res);
            this.table = res.data;
          }else {
              this.$message.error(res.msg);
          }
        });
      },
      /**
       * 搜索按钮
       */
      getTableConfigListBtn: function () {
        this.getTableConfigList();
      },
      /**
       * 新增按钮——打开dialog
       */
      TableConfigAddBtn: function () {
        this.$refs.dialog.TableConfigDialog();
      },
      /**
       * 编辑——打开dialog
       */
      editTableConfigBtn: function (index, row) {
        let para = Object.assign({}, row);
        this.$refs.dialog.TableConfigDialog(para);
      },
      /**
       * table列表中，测试、删除*
       */
      testTableConfigBtn: function (index, row) {
        this.$confirm('对' + row.name + '进行同步测试？', '提示', {}).then(() => {
          let para = {
            id: row.id,
          };
          this.tableLoading=true;
          testTableSyn(para).then((res) => {
            this.tableLoading=false;
            if (res.code === '0') {
              this.$message.success(res.msg);
            } else {
              this.$message.error(res.msg);
            }
            this.getTableConfigList();
          });
        }).catch(() => {
          this.tableLoading=false;
        });
      },
      deleteTableConfigBtn: function (index, row) {
        this.$confirm('确认删除吗？', '提示', {}).then(() => {
          let para = {
            id: row.id,
          };
          this.tableLoading=true;
          deleteTableConfig(para).then((res) => {
            this.tableLoading=false;
            if (res.code === '0') {
              this.$message.success(res.msg);
              this.getTableConfigList();
            } else {
              this.$message.error(res.msg);
            }
          });
        }).catch(() => {
          //取消操作
        });
      },
      /**
       * 读取数据时，将json转换为可以解析的数组格式
       * 还有true、false等转换
       */
      jsonToArray: function (res) {
        for (let i = 0; i < res.data.length; i++) {
          res.data[i].jointable = JSON.parse(res.data[i].jointable);
          res.data[i].relation = JSON.parse(res.data[i].relation);
          res.data[i].columnto = JSON.parse(res.data[i].columnto);
          res.data[i].columnfrom = JSON.parse(res.data[i].columnfrom);
          res.data[i].synvalue = JSON.parse(res.data[i].synvalue);
          if ("true" === res.data[i].logflag) {
            res.data[i].logflag = true;
          }
          if ("true" === res.data[i].errorflag) {
            res.data[i].errorflag = true;
          }
          if ("true" === res.data[i].synflag) {
            res.data[i].synflag = true;
          }
        }
      },
    },
    mounted:function () {
      this.getTableConfigList();
    }
  }
</script>

<style>

</style>

