<template>
  <div class="index" v-loading.fullscreen.lock="fullscreenLoading" element-loading-text="拼命加载中...">
    <input type="file" @change="importFile(this)" id="imFile" style="display: none"
           accept="application/vnd.openxmlformats-officedocument.spreadsheetml.sheet, application/vnd.ms-excel"/>
    <a id="downlink"></a>
    <el-button class="button" @click="downloadFile(excelData)">导出</el-button>
    <!--错误信息提示-->
    <el-dialog title="提示" v-model="errorDialog" size="tiny">
      <span>{{errorMsg}}</span>
      <span slot="footer" class="dialog-footer">
    <el-button type="primary" @click="errorDialog=false">确认</el-button>
  </span>
    </el-dialog>
    <!--展示导入信息-->
    <el-table stripe :data="excelData" tooltip-effect="dark">
      <el-table-column align="center" label="系统" prop="xt" show-overflow-tooltip></el-table-column>
      <el-table-column align="center" label="时间" prop="sj" show-overflow-tooltip></el-table-column>
      <el-table-column align="center" label="周一" prop="zy" show-overflow-tooltip>
        <template slot-scope="scope">
          <span v-bind:style="{ color: scope.row.zy==='异常'?'red':'' }" >{{ scope.row.zy }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="周二" prop="zr" show-overflow-tooltip>
        <template slot-scope="scope">
          <span v-bind:style="{ color: scope.row.zr==='异常'?'red':'' }" >{{ scope.row.zr }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="周三" prop="zs" show-overflow-tooltip>
        <template slot-scope="scope">
          <span v-bind:style="{ color: scope.row.zs==='异常'?'red':'' }" >{{ scope.row.zs }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="周四" prop="zss" show-overflow-tooltip>
        <template slot-scope="scope">
          <span v-bind:style="{ color: scope.row.zss==='异常'?'red':'' }" >{{ scope.row.zss }}</span>
        </template>
      </el-table-column>
      <el-table-column align="center" label="周五" prop="zw" show-overflow-tooltip>
        <template slot-scope="scope">
          <span v-bind:style="{ color: scope.row.zw==='异常'?'red':'' }" >{{ scope.row.zw }}</span>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
  // 引入xlsx
  var XLSX = require('xlsx')
  import axios from 'axios'
  export default {
    name: 'Syjl',
    data() {
      return {
        fullscreenLoading: false, // 加载中
        outFile: '',  // 导出文件el
        errorDialog: false, // 错误信息弹窗
        errorMsg: '', // 错误信息内容
        excelData: [  // 测试数据
          {
            xt: '', sj: '', zy: '', zr: '', zs: '',zss:'',zw:''
          },
        ]
      }
    },
    mounted() {
      this.imFile = document.getElementById('imFile')
      this.outFile = document.getElementById('downlink')
    },
    created(){
      axios.get('http://192.168.0.124:8080/xcb/syxcb')
        .then(res=>{
          this.excelData=res.data;
        })
    },
    methods: {
      downloadFile: function (rs) { // 点击导出按钮
        let data = [{}]
        for (let k in rs[0]) {
          data[0][k] = k
        }
        data = data.concat(rs)
        this.downloadExl(data, '信息系统巡检表')
      },
      downloadExl: function (json, downName, type) {  // 导出到excel
        let keyMap = [] // 获取键
        for (let k in json[0]) {
          keyMap.push(k)
        }
        console.info('keyMap', keyMap, json)
        let tmpdata = [] // 用来保存转换好的json
        json.map((v, i) => keyMap.map((k, j) => Object.assign({}, {
          v: v[k],
          position: (j > 25 ? this.getCharCol(j) : String.fromCharCode(65 + j)) + (i + 1)
        }))).reduce((prev, next) => prev.concat(next)).forEach(function (v) {
          tmpdata[v.position] = {
            v: v.v
          }
        })
        let outputPos = Object.keys(tmpdata)  // 设置区域,比如表格从A1到D10
        let tmpWB = {
          SheetNames: ['mySheet'], // 保存的表标题
          Sheets: {
            'mySheet': Object.assign({},
              tmpdata, // 内容
              {
                '!ref': outputPos[0] + ':' + outputPos[outputPos.length - 1] // 设置填充区域
              })
          }
        }
        let tmpDown = new Blob([this.s2ab(XLSX.write(tmpWB,
          {bookType: (type === undefined ? 'xlsx' : type), bookSST: false, type: 'binary'} // 这里的数据是用来定义导出的格式类型
        ))], {
          type: ''
        })  // 创建二进制对象写入转换好的字节流
        var href = URL.createObjectURL(tmpDown)  // 创建对象超链接
        this.outFile.download = downName + '.xlsx'  // 下载名称
        this.outFile.href = href  // 绑定a标签
        this.outFile.click()  // 模拟点击实现下载
        setTimeout(function () {  // 延时释放
          URL.revokeObjectURL(tmpDown) // 用URL.revokeObjectURL()来释放这个object URL
        }, 100)
      },
      analyzeData: function (data) {  // 此处可以解析导入数据
        return data
      },
      s2ab: function (s) { // 字符串转字符流
        var buf = new ArrayBuffer(s.length)
        var view = new Uint8Array(buf)
        for (var i = 0; i !== s.length; ++i) {
          view[i] = s.charCodeAt(i) & 0xFF
        }
        return buf
      },
      getCharCol: function (n) { // 将指定的自然数转换为26进制表示。映射关系：[0-25] -> [A-Z]。
        let s = ''
        let m = 0
        while (n > 0) {
          m = n % 26 + 1
          s = String.fromCharCode(m + 64) + s
          n = (n - m) / 26
        }
        return s
      },
      fixdata: function (data) {  // 文件流转BinaryString
        var o = ''
        var l = 0
        var w = 10240
        for (; l < data.byteLength / w; ++l) {
          o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w, l * w + w)))
        }
        o += String.fromCharCode.apply(null, new Uint8Array(data.slice(l * w)))
        return o
      }
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .el-table th > .cell {
    text-align: center;
  }

  .button {
    margin-bottom: 20px;
  }
</style>
