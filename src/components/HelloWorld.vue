<template>
  <div style="text-align:left;">
    <el-button type="primary" @click="dialogFormVisible = true;" style="margin-bottom: 5px">新建周表</el-button>
    <el-button type="primary" @click="syjl" style="margin-bottom: 5px">所有记录</el-button>
    <h2>当前表时间：{{this.sj}}</h2>
    <el-card class="box-card">
      <el-table
        stripe
        :data="tableData"
        style="width: 100%">
        <el-table-column
          label="系统"
          width="180">
          <template slot-scope="scope">
            <span  >{{ scope.row.xt }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="周一"
          width="180">
          <template slot-scope="scope">
            <span v-bind:style="{ color: scope.row.zy==='异常'?'red':'' }" >{{ scope.row.zy }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="周二"
          width="180">
          <template slot-scope="scope">
            <span v-bind:style="{ color: scope.row.zr==='异常'?'red':'' }">{{ scope.row.zr }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="周三"
          width="180">
          <template slot-scope="scope">
            <span v-bind:style="{ color: scope.row.zs==='异常'?'red':'' }">{{ scope.row.zs }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="周四"
          width="180">
          <template slot-scope="scope">
            <span v-bind:style="{ color: scope.row.zss==='异常'?'red':'' }">{{ scope.row.zss }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="周五"
          width="180">
          <template slot-scope="scope">
            <span v-bind:style="{ color: scope.row.zw==='异常'?'red':'' }">{{ scope.row.zw }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button
              size="mini"
              @click="handleEdit(scope.$index, scope.row)">编辑
            </el-button>

          </template>
        </el-table-column>
      </el-table>
    </el-card>


    <el-dialog title="新建周表" :visible.sync="dialogFormVisible">
      <el-date-picker
        v-model="xjkssj"
        value-format="yyyy:MM:dd"
        placeholder="开始日期">
      </el-date-picker>
      <span>---</span>
      <el-date-picker
        v-model="xjjssj"
        value-format="yyyy:MM:dd"
        placeholder="结束日期">
      </el-date-picker>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="qdxjzb">确 定</el-button>
      </div>
    </el-dialog>

    <el-dialog title="编辑" :visible.sync="dialogFormVisible2">
      <el-form :model="dq">
        <el-form-item label="周一：" :label-width="formLabelWidth">
          <el-select  v-model="dq.zy" >
            <el-option
              label="正常"
              value="正常">
            </el-option>
            <el-option
              label="异常"
              value="异常">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="周二：" :label-width="formLabelWidth">
          <el-select  v-model="dq.zr" >
            <el-option
              label="正常"
              value="正常">
            </el-option>
            <el-option
              label="异常"
              value="异常">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="周三：" :label-width="formLabelWidth">
          <el-select  v-model="dq.zs" >
            <el-option
              label="正常"
              value="正常">
            </el-option>
            <el-option
              label="异常"
              value="异常">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="周四：" :label-width="formLabelWidth">
          <el-select  v-model="dq.zss" >
            <el-option
              label="正常"
              value="正常">
            </el-option>
            <el-option
              label="异常"
              value="异常">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="周五：" :label-width="formLabelWidth">
          <el-select  v-model="dq.zw" >
            <el-option
              label="正常"
              value="正常">
            </el-option>
            <el-option
              label="异常"
              value="异常">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="qxxg">取 消</el-button>
        <el-button type="primary" @click="bjqd">确 定</el-button>
      </div>
    </el-dialog>

  </div>
</template>

<script>
  import axios from 'axios';

  export default {
    name: 'HelloWorld',
    data() {
      return {
        dialogFormVisible: false,
        dialogFormVisible2: false,
        sj: ' ',
        dq:{},
        tableData: [],
        xjkssj: '',
        xjjssj: '',
        formLabelWidth: '120px'
      }
    },
    created() {
      this.zxsj()
    },
    methods: {
      qxxg(){//取消修改
        this.zxsj()
        this.dq={};
        this.dialogFormVisible2 = false
      },

      syjl(){//所以记录
        //window.location.href = '/#/Syjl'
        window.open('/#/Syjl')
      },

      bjqd(){
        this.xgxcb()
        this.dialogFormVisible2=false;
      },

      xgxcb(){//修改记录
        axios.post("http://192.168.0.124:8080/xcb/xgxcb",this.dq)
      },
      zxsj() {//最新记录
        axios.get("http://192.168.0.124:8080/xcb/zxxcb")
          .then((res) => {
            this.tableData = res.data;
            this.sj=res.data[0].sj;
            console.log(res)
          })
      },
      qdxjzb() {//确定新建
        this.sj = this.xjkssj + "-" + this.xjjssj,
          axios.get('http://192.168.0.124:8080/xcb/xjxcb', {
            params: {
              xjkssj: this.xjkssj,
              xjjssj: this.xjjssj
            }
          })
            .then(function (response) {
              console.log(response);
            })
            .catch(function (error) {
              console.log(error);
            });
        for (let i = 0; i < 11; i++) {
          this.tableData[i].zy = ''
          this.tableData[i].zr = ''
          this.tableData[i].zs = ''
          this.tableData[i].zss = ''
          this.tableData[i].zw = ''
        }
        this.dq={};
        this.zxsj();
        this.dialogFormVisible = false;
      },
      handleEdit(index, row) {
        this.dq=row;
        this.dialogFormVisible2=true;
      },
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
