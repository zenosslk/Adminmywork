<template>
  <div class="dataAnalysis">
    <!-- 报表头部区域 -->
    <div class="dataAnalysis_head">
      <div class="dataAnalysis_tit">
        <span>信息报表</span>
        <img class="dataAnalysis_icon" src="../../../assets/icon/mark.png" alt>
      </div>
      <!-- 搜索图表 -->
      <div class="dataAnalysis_sea">
        <el-input placeholder="搜索图表" prefix-icon="el-icon-search" v-model="input2"></el-input>
      </div>
      <!-- 新建 -->
      <div class="dataAnalysis_setting">
        <span></span>
        <el-dropdown>
          <el-button type="primary">
            新建
            <i class="el-icon-arrow-down el-icon--right"></i>
          </el-button>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item>新建报表</el-dropdown-item>
            <el-dropdown-item>新建统计图</el-dropdown-item>
            <el-dropdown-item>新建报表分类</el-dropdown-item>
            <el-dropdown-item>新建全部报表</el-dropdown-item>
            <el-dropdown-item>新建全部统计图</el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
    </div>

    <!-- 报表内容主体区域 -->
    <el-col :span="24" style="padding-bottom:0px">
      <el-form :model="filters" :inline="true">
        <el-form-item>
          <el-input placeholder="姓名" v-model="select_word"></el-input>
        </el-form-item>

        <!-- <el-form-item>
          <el-button type="primary">查询</el-button>
        </el-form-item>-->

        <el-form-item>
          <el-button type="primary" @click="useradd">新增</el-button>
        </el-form-item>

        <el-form-item>
          <el-button type="primary" icon="delete" @click="deleteAll">批量删除</el-button>
        </el-form-item>
      </el-form>
    </el-col>

    <!-- 内容区域 -->
    <div class="dataAnalysis_main">
      <div class="oneData">
        <el-table
          class="table"
          @select="select"
          @select-all="select"
          :data="data.slice((currentPage-1)*pagesize,currentPage*pagesize)"
          border
          ref="multipleTable"
          @selection-change="handleSelectionChange"
          row-key="id"
        >
          <!-- check选择框 -->
          <el-table-column type="selection" width="55" align="center"></el-table-column>
          <!-- 带文件夹信息 -->
          <el-table-column prop="name" label="人物" sortable width="150"></el-table-column>
          <el-table-column prop="date" label="时间" sortable width="200"></el-table-column>
          <el-table-column prop="address" label="描述" sortable></el-table-column>
          <!-- ---操作页面--- -->
          <el-table-column label="操作" width="180" align="center">
            <template slot-scope="scope">
              <el-button
                type="text"
                icon="el-icon-edit"
                @click="handleEdit(scope.$index, scope.row)"
              >编辑</el-button>
              <el-button
                type="text"
                icon="el-icon-delete"
                @click="handleDelete(scope.$index, scope.row)"
              >删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>

    <!-- //////////////////////新增/// 新增弹出框////////////////////////////// -->
    <el-dialog v-el-drag-dialog title="新增用户" :visible.sync="addVisible" width="50%">
      <el-form :model="addfromm" ref="addfromm">
        <el-form-item label="信息">
          <el-input v-model="addfromm.csentence" placeholder="信息"></el-input>
        </el-form-item>
        <el-form-item label="人物">
          <el-input v-model="addfromm.name"></el-input>
        </el-form-item>

        <el-form-item label="时间" style="display: inline-block;">
          <el-date-picker
            v-model="addfromm.date"
            type="date"
            placeholder="选择日期时间"
            value-format=" yyyy-MM-dd HH:mm"
            format="yyyy-MM-dd HH:mm"
          ></el-date-picker>
        </el-form-item>

        <el-form-item label="描述">
          <el-input v-model="addfromm.address" type="textarea"></el-input>
        </el-form-item>
      </el-form>
      <!-- 进行确认事件操作  -->
      <span slot="footer" class="dialog-footer">
        <el-button @click.native="addVisible = false">取 消</el-button>
        <el-button type="primary" @click.native="saveAddfrom">提交</el-button>
      </span>
    </el-dialog>
    <!-- //////////////////////编辑弹出框 ////////////////////-->
    <el-dialog v-el-drag-dialog title="编辑" :visible.sync="editVisible" width="30%">
      <el-form ref="form" :model="from" label-width="100px">
        <!-- 调整日期 -->
        <el-form-item label="日期">
          <el-date-picker
            type="datetime"
            placeholder="选择日期"
            value-format="yyyy-MM-dd"
            v-model="from.date"
            style="width: 100%;"
          ></el-date-picker>
        </el-form-item>
        <!-- 调整修改人 -->
        <el-form-item class="needmodify" label="修改人">
          <el-input v-model="from.name"></el-input>
        </el-form-item>
        <!-- 调整描述 -->
        <el-form-item label="修改地址">
          <el-input v-model="from.address"></el-input>
        </el-form-item>

        <!-- 进行确认事件 -->
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click.native="saveEdit">确 定</el-button>
      </span>
    </el-dialog>

    <!-- ////////////////删除提示框/////////////////// -->
    <el-dialog v-el-drag-dialog title="提示" :visible.sync="delVisible" width="300px" center>
      <div class="del-dialog-cnt">删除不可恢复，是否确定删除？</div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="delVisible = false">取 消</el-button>
        <el-button type="primary" @click="deleteRow">确 定</el-button>
      </span>
    </el-dialog>
    <!-- //////////分页---//////////------------------ -->
    <el-footer>
      <template>
        <div class="block">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="currentPage"
            :page-sizes="[5, 10, 20, 40]"
            :page-size="pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="data.length"
          ></el-pagination>
        </div>
      </template>
    </el-footer>
  </div>
</template>


<script>
import elDragDialog from "@/directive/el-drag-dialog"; // base on element-ui
export default {
  //分页数据
  directives: { elDragDialog },
  data() {
    return {
      total: "0",
      choseData: [],
      currentPage: 1,
      select_word: "",
      pagesize: 10,
      show: true,
      editVisible: false,
      delVisible: false,
      addVisible: false,
      tableData1: [],
      del_list: [],
      new_list: [],
      input2: "",
      filters: {
        name: ""
      },
      from: {
        csentence: "",
        name: "",
        date: "",
        address: ""
      },
      addfromm: {
        csentence: "",
        name: "",
        date: "",
        address: ""
      }
    };
  },
  params: {
    page: 1
  },

  mounted() {
    this.gettableData();
  },
  computed: {
    data() {
      return this.tableData1.filter(item => {
        // console.log(item)
        let is_del = false;
        for (let i = 0; i < this.del_list.length; i++) {
          if (item.name === this.del_list[i].name) {
            is_del = true;
            break;
          }
        }
        if (!is_del) {
          if (
            item.name.indexOf(this.select_word) > -1 ||
            item.address.indexOf(this.select_word) > -1
          ) {
            return item;
          }
        }
      });
    }
  },
  methods: {
    // 新增用户
    useradd() {
      this.addVisible = true;
      this.addfromm = {
        csentence: "",
        name: "",
        date: "",
        address: ""
      };
    },
    // 新增用户点击确定操作
    saveAddfrom() {
      this.$refs.addfromm.validate(valid => {
        if (valid) {
          this.addVisible = false;
          this.data.pop();
          this.data.unshift(this.addfromm);
          this.$message.success("添加成功");
        }
      });
    },

    gettableData() {
      this.$fetch("/userDetail", {}).then(response => {
        if (response.state == 0) {
          this.tableData1 = response.data.result;
        }
      });
    },
    // 分页---------------------------
    // 初始页currentPage、初始每页数据数pagesize和数据data
    handleSizeChange: function(size) {
      this.pagesize = size;
    },
    handleCurrentChange: function(currentPage) {
      this.currentPage = currentPage;
    },
    select(selection) {
      this.choseData = selection;

      if (selection.length == 0) {
        this.show = true;
      } else {
        this.show = false;
      }
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    },
    // 点击删除
    handleDelete(index, row) {
      console.log(index);
      this.idx = index;
      this.delVisible = true;
    },
    // 点击编辑
    handleEdit(index, row) {
      this.idx = index;
      const item = this.data[index];
      this.from = {
        csentence: item.csentence,
        name: item.name,
        data: item.date,
        address: item.address
      };
      this.editVisible = true;
    },
    // 保存编辑
    saveEdit() {
      this.$set(this.data, this.idx, this.from);
      this.editVisible = false;
      this.$message.success(`修改第 ${this.idx + 1} 行成功`);
    },
    // 确定删除
    deleteRow() {
      this.data.splice(this.idx, 1);
      this.$message.success("删除成功");
      this.delVisible = false;
    },
    // 全部删除
    deleteAll() {
      console.log(this.choseData.length);
      if (this.choseData.length == 0) {
        this.$message.warning("请选择删除目标");
      } else {
        this.$message.success("删除成功");
      }
      this.del_list = this.del_list.concat(this.choseData);
      this.choseData = [];
    }
  }
};
</script>

<style scoped>
/* 报表头部区域 */
.dataAnalysis {
  background-color: #fff;
}
.dataAnalysis_sea {
  display: flex;
  align-items: center;
}
.dataAnalysis_head {
  font-size: 18px;
  box-sizing: border-box;
  height: 60px;
  /* line-height: 60px; */
  width: 100%;
  overflow: hidden;
  font-weight: 500;
  padding: 0 20px;
  background-color: #fff;
  position: relative;
  border-bottom: 1px solid #d8dde8;
  display: flex;
  justify-content: space-between;
}
.dataAnalysis_tit {
  line-height: 60px;
}
.dataAnalysis_icon {
  width: 18px;
  height: 18px;
}
.needmodify {
  margin-top: 10px;
  margin-bottom: 10px !important;
}
/* 修改input样式 */
.el-input {
  position: relative;
  /* top: 10px; */
}
.dataAnalysis_setting {
  display: flex;
  justify-content: center;
  align-items: center;
}

/* 报表二级头部区域 */
.dataAnalysis_content {
  display: flex;
  height: 40px;
  width: 100%;
  background-color: #fff;
  padding: 10px 0 0 20px;
  margin: 0;
  border: 0;
  line-height: 140%;
  color: #999;
  line-height: 36px;
  /* font: 13px/1.5 tahoma, arial, "Hiragino Sans GB", "Hiragino Sans GB W3",
    "Microsoft Yahei", \5b8b\4f53; */
}
.bi-detail__select__label {
  margin-right: 10px;
  height: 28px;
  float: left;
}
.el-col {
  padding-left: 55px;
  display: flex;
  align-items: center;
  height: 50px;
}
.el-form-item--small.el-form-item {
  margin-bottom: 0;
}

.dataAnalysis_spe {
  margin-left: 10px;
  margin-right: 10px;
}
.dataAnalysis_chart {
  margin-right: 10px;
}
/* .el-button--primary {
  background-color: #fff;
  color: #ccc;
  border: 1px solid #ccc;
} */
.dataAnalysis_need {
  position: relative;
  top: -3px;
}
.dataAnalysis_batch {
  margin-left: 10px;
  margin-right: 10px;
}
/* .el-form-item el-form-item--small{
  margin-bottom: 10px;
} */
.dataAnalysis_delete {
  margin-right: 10px;
  /* display: inline;
  border-radius: 10%;
  background-color: #fff;
  color: #666;
  border: 1px solid #ddd;
  cursor: pointer;
  margin-top: 2px; */
}
.dataAnalysis_export {
  margin-right: 10px;
}
.del-dialog-cnt {
  text-align: center;
}
handle-select {
  width: 120px;
}

.handle-input {
  width: 300px;
  display: inline-block;
}
.del-dialog-cnt {
  font-size: 16px;
  text-align: center;
}
.table {
  width: 100%;
  font-size: 14px;
}
.red {
  color: #ff0000;
}
.mr10 {
  margin-right: 10px;
}
</style>
