<template>
  <div>
    <el-form :inline="true" :model="searchForm">
      <el-form-item label="规划设计名称">
        <el-input v-model="searchForm.planDesignName" placeholder="规划设计名称" style="width: 200px"></el-input>
      </el-form-item>
      <el-form-item label="规划类型">
        <el-select v-model="searchForm.specId" placeholder="规划类型">
          <el-option v-once v-for="(item,index) in this.planDesignType.entries" :key="item.key" :value="item.value"
                     :label="item.text"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="设计人">
        <el-input v-model="searchForm.designer" placeholder="设计人" style="width: 200px"></el-input>
      </el-form-item>
      <el-form-item label="设计时间">
        <el-date-picker
            v-model="searchForm.designTime"
            type="datetimerange"
            range-separator="至"
            start-placeholder="开始日期"
            end-placeholder="结束日期"
            align="right">
        </el-date-picker>
      </el-form-item>
    </el-form>
    <el-button type="primary" icon="el-icon-search" @click="search">查询</el-button>
    <el-button type="primary" icon="el-icon-plus" @click="getBillNo(),dialogFormVisible = true">新 增</el-button>

    <el-table
        :data="planDesignInfoList"
        border
        style="width: 100%">
      <el-table-column
          prop="plan_bill_no"
          label="规划工单号"
          width="220">
      </el-table-column>
      <el-table-column
          prop="plan_design_name"
          label="规划设计名称"
          width="220">
      </el-table-column>
      <el-table-column
          prop="design_company"
          label="设计单位"
          width="230">
      </el-table-column>
      <el-table-column
          prop="spec_id"
          label="业务类型"
          :formatter="formatterDesignType"
          width="120">
      </el-table-column>
      <el-table-column
          prop="project_director"
          label="项目总负责人"
          width="120">
      </el-table-column>
      <el-table-column
          prop="spec_leader"
          label="专业负责人"
          width="120">
      </el-table-column>
      <el-table-column
          prop="designer"
          label="设计人"
          width="120">
      </el-table-column>
      <el-table-column
          prop="reviewer"
          label="校审人"
          width="120">
      </el-table-column>
      <el-table-column
          prop="create_time"
          label="设计新建时间"
          width="170">
      </el-table-column>
      <el-table-column
          label="操作"
          width="100">
        <template slot-scope="scope">
          <el-button @click="handleClick(scope.row)" type="text" size="small">查看</el-button>
          <el-button type="text" size="small">编辑</el-button>
        </template>
      </el-table-column>
    </el-table>

    <el-pagination
        @next-click="next"
        @prev-click="previous"
        @current-change="handleCurrentChange"
        :current-page="Number(this.searchForm.current)"
        :page-sizes="[1,2,3]"
        :page-size="1"
        layout="total, sizes, prev, pager, next, jumper"
        :total="this.total">
    </el-pagination>


    <el-dialog title="新增规划设计评估记录" :visible.sync="dialogFormVisible" width="1410px">
      <el-form :model="form">
        <el-form-item label="规划工单编号" :label-width="formLabelWidth">
          <el-input v-model="form.planBillNo" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="规划设计名称" :label-width="formLabelWidth">
          <el-input v-model="form.planDesignName" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="设计单位" :label-width="formLabelWidth">
          <el-input v-model="form.designCompany" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="规划类型" :label-width="formLabelWidth">
          <el-input v-model="form.specId" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="项目总负责人" :label-width="formLabelWidth">
          <el-input v-model="form.projectDirector" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="专业负责人" :label-width="formLabelWidth">
          <el-input v-model="form.specLeader" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="设计人" :label-width="formLabelWidth">
          <el-input v-model="form.designer" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="校审人" :label-width="formLabelWidth">
          <el-input v-model="form.reviewer" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div style="display: flex; align-items: center;">
        <el-upload
            class="upload-demo"
            action="http://localhost:8080/upload.do"
            :on-remove="handleRemove"
            :before-remove="beforeRemove"
            :limit="1"
            :accept="'.dwg'"
            :on-exceed="exceedTips"
            :on-success="handleDwgSuccess1">
          <span style="margin-right: 10px;">系统规划CAD图纸：</span>
          <el-button size="small" type="primary">点击上传</el-button>
          <div slot="tip" class="el-upload__tip">只能上传dwg格式文件,不超过10MB</div>
        </el-upload>
        <div style="margin-left: 200px;"></div>
        <el-upload
            class="upload-demo"
            action="http://localhost:8080/upload.do"
            :on-remove="handleRemove"
            :before-remove="beforeRemove"
            :limit="1"
            :accept="'.xls'"
            :on-exceed="exceedTips"
            :on-success="handleDwgSuccess2">
          <span style="margin-right: 10px;">系统规划Excel文件：</span>
          <el-button size="small" type="primary">点击上传</el-button>
          <div slot="tip" class="el-upload__tip">只能上传excel格式文件,不超过10MB</div>
        </el-upload>
        <div style="margin-left: 200px;"></div>
        <el-upload
            class="upload-demo"
            action="http://localhost:8080/upload.do"
            :on-remove="handleRemove"
            :before-remove="beforeRemove"
            :limit="1"
            :accept="'.xls'"
            :on-exceed="exceedTips"
            :on-success="handleDwgSuccess3">
          <span style="margin-right: 10px;">波道规划Excel文件：</span>
          <el-button size="small" type="primary">点击上传</el-button>
          <div slot="tip" class="el-upload__tip">只能上传excel格式文件,不超过10MB</div>
        </el-upload>
      </div>
      CAD图纸关键坐标（X,Y）: 左上坐标：
      <el-input v-model="input1" placeholder="（X,Y）"></el-input>
      右下坐标：
      <el-input v-model="input2" placeholder="（X,Y）"></el-input>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">关 闭</el-button>
        <el-button type="primary" @click="save">保 存</el-button>
        <el-button type="primary" @click="saveAndAnalyse">保存并分析</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
import axios from 'axios'
import {planDesignType} from '@/enum.js'

export default {
  name: 'PlanDesign',
  data() {
    return {
      planDesignType,
      planDesignInfoList: [],
      formLabelWidth: '120px',
      form: {
        planBillNo: '',
        planDesignName: '',
        designCompany: '',
        specId: '',
        projectDirector: '',
        specLeader: '',
        designer: '',
        reviewer: '',
      },
      searchForm: {
        planDesignName: "",
        specId: 1,
        designer: "",
        designTime: [],
        current: 1,
        size: 1
      },
      total: 0,
      dialogFormVisible: false,
      dwgFileUrl: "",
      xls1FileUrl: '',
      xls2FileUrl: '',
      input1: '',
      input2: '',
      coordinates: '',
    }
  },
  methods: {

    // 新增一个《格式化业务类型》方法
    formatterDesignType(row) {
      if(row.spec_id==1){
        return "OTN业务"
      }else if(row.spec_id==2){
        return "网元业务"
      }
    },
    search() {
      let _this = this;
      axios.post('http://localhost:8080/searchBill.do', this.searchForm)
          .then(res => {
            console.log("服务器返回的结果：", res);
            _this.planDesignInfoList = res.data.data.records;
            this.total = res.data.data.total;
          }, err => {
            console.log("服务器出错，err代表服务器返回的错误信息");
          })
    },
    previous(val) {
      console.log("向前翻页", `${val}`);
      this.searchForm.current = `${val}`;
      let _this = this;
      axios.post("http://localhost:8080/searchBill.do", this.searchForm)
          .then(res => {
            console.log("res是服务器返回的结果", res);
            _this.planDesignInfoList = res.data.data.records;
            this.total = res.data.data.total;
          }, err => {
            console.log("服务器出错，err代表服务器返回的错误信息");
          });
    },
    next(val) {
      console.log("向下翻页", `${val}`)
      this.searchForm.current = `${val}`;
      let _this = this;
      axios.post('http://localhost:8080/searchBill.do', this.searchForm)
          .then(res => {
            console.log("res是服务器返回的结果", res);
            _this.planDesignInfoList = res.data.data.records;
            this.total = res.data.data.total;
          }, err => {
            console.log("服务器出错，err代表服务器返回的错误信息")
          });
    },
    handleCurrentChange(val) {
      console.log(`当前页: ${val}`);
      this.searchForm.current = `${val}`;
      let _this = this;
      axios.post('http://localhost:8080/searchBill.do', this.searchForm)
          .then(res => {
            console.log("res是服务器返回的结果", res);
            _this.planDesignInfoList = res.data.data.records;
            this.total = res.data.data.total;
          }, err => {
            console.log("服务器出错，err代表服务器返回的错误信息")
          });
    },
    getBillNo() {
      // 用axios对后端发请求获取数据
      let _this = this;
      axios.get('http://localhost:8080/getBillNo.do')
          .then(function (response) {
            // 处理成功情况
            _this.form.planBillNo = response.data.data;
            console.log("BillNo：", _this.planBillNo)
          })
          .catch(function (error) {
            // 处理错误情况
            console.log(error);
          })
          .then(function () {
            // 总是会执行
          });
    },
    save() {
      /*
      跟着《需求》走，调用后台的xxx方法
       */
      // 向给定ID的用户发起请求
      let _this = this; //axios中的this被转化了，并不是vue的实例
      const str1 = this.input1;
      const str2 = this.input2;
      const index = str1.indexOf(",");
      const index1 = str2.indexOf(",");
      const index2 = str1.indexOf(")")
      const index3 = str2.indexOf(")");
      const cad_coord_left = str1.substring(1, index);
      const cad_coord_top = str1.substring(index + 1, index2)
      const cad_coord_right = str2.substring(1, index1);
      const cad_coord_bottom = str2.substring(index1 + 1, index3)
      axios.post('http://localhost:8080/createBill.do', {
        "plan_bill_no": this.form.planBillNo,
        "plan_design_name": this.form.planDesignName,
        "design_company": this.form.designCompany,
        "spec_id": this.form.specId,
        "project_director": this.form.projectDirector,
        "spec_leader": this.form.specLeader,
        "designer": this.form.designer,
        "reviewer": this.form.reviewer,
        "system_cad_file_name": this.dwgFileName,
        "system_cad_file_url": this.dwgFileUrl,
        "system_excel_file_name": this.xls1FileName,
        "system_excel_file_url": this.xls2FileName,
        "channel_excel_file_name": this.xls2FileName,
        "channel_excel_file_url": this.xls2FileUrl,
        "cad_coord_left": cad_coord_left,
        "cad_coord_top": cad_coord_top,
        "cad_coord_right": cad_coord_right,
        "cad_coord_bottom": cad_coord_bottom
      })
          .then(function (response) {
            console.log(response);
            /*
            如果保存成功，则关闭Dialog，并且提示message
            如果保存失败，则保持Dialog，并且提示message
             */
            if (response.data.code == 200) {
              _this.dialogFormVisible = false;//关闭Dialog
              _this.$message({
                message: response.data.message,
                type: 'success'
              });
            } else {
              _this.$message.error(response.data.message);
            }
          })
          .catch(function (error) {
            // 处理错误情况
            console.log(error);
          })
          .then(function () {
            // 总是会执行
          });
    },
    saveAndAnalyse() {
      let _this = this; //axios中的this被转化了，并不是vue的实例
      const str1 = this.input1;
      const str2 = this.input2;
      const index = str1.indexOf(",");
      const index1 = str2.indexOf(",");
      const index2 = str1.indexOf(")")
      const index3 = str2.indexOf(")");
      const cad_coord_left = str1.substring(1, index);
      const cad_coord_top = str1.substring(index + 1, index2)
      const cad_coord_right = str2.substring(1, index1);
      const cad_coord_bottom = str2.substring(index1 + 1, index3)
      axios.post('http://localhost:8080/createBillAndAnalyse.do', {
        "plan_bill_no": this.form.planBillNo,
        "plan_design_name": this.form.planDesignName,
        "design_company": this.form.designCompany,
        "spec_id": this.form.specId,
        "project_director": this.form.projectDirector,
        "spec_leader": this.form.specLeader,
        "designer": this.form.designer,
        "reviewer": this.form.reviewer,
        "system_cad_file_name": this.dwgFileName,
        "system_cad_file_url": this.dwgFileUrl,
        "system_excel_file_name": this.xls1FileName,
        "system_excel_file_url": this.xls2FileName,
        "channel_excel_file_name": this.xls2FileName,
        "channel_excel_file_url": this.xls2FileUrl,
        "cad_coord_left": cad_coord_left,
        "cad_coord_top": cad_coord_top,
        "cad_coord_right": cad_coord_right,
        "cad_coord_bottom": cad_coord_bottom
      })
          .then(function (response) {
            console.log(response);
            /*
            如果保存成功，则关闭Dialog，并且提示message
            如果保存失败，则保持Dialog，并且提示message
             */
            if (response.data.code == 200) {
              _this.dialogFormVisible = false;//关闭Dialog
              _this.$message({
                message: response.data.message,
                type: 'success'
              });
            } else {
              _this.$message.error(response.data.message);
            }
          })
          .catch(function (error) {
            // 处理错误情况
            console.log(error);
          })
          .then(function () {
            // 总是会执行
          });
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    beforeRemove(file, fileList) {
      //删除之前要问一下用户
      return this.$confirm(`确定移除 ${file.name}？`);
    },
    handleDwgSuccess1(res) {
      this.dwgFileUrl = res.data[0];
      console.log("上传成功后，服务器返回的文件的url（地址）：", this.dwgFileUrl);
      this.dwgFileName = this.dwgFileUrl.substr(29);
      console.log("文件名称：", this.dwgFileName);
    },
    handleDwgSuccess2(res) {
      this.xls1FileUrl = res.data[0];
      console.log("上传成功后，服务器返回的文件的url（地址）：", this.xls1FileUrl);
      this.xls1FileName = this.xls1FileUrl.substr(29);
      console.log("文件名称：", this.xls1FileName);
    },
    handleDwgSuccess3(res) {
      this.xls2FileUrl = res.data[0];
      console.log("上传成功后，服务器返回的文件的url（地址）：", this.xls2FileUrl);
      this.xls2FileName = this.xls2FileUrl.substr(29);
      console.log("文件名称：", this.xls2FileName);
    },
    exceedTips() {
      this.$message.error("只能上传一个文件");
    }
  },

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.el-form-item {
  display: inline-block !important;
}

.el-input {
  width: 450px;
}
</style>
