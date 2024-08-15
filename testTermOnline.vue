<template>
  <div>
      <el-row :gutter="20" style="height: 20%">
        <el-col :span="4" style="margin-top: 20px;">
          <div style="font-size: 20px;font-weight: bold">集中器总数：<span style="font-size:20px;color: #ff5100">{{this.termSum}}</span>
          </div>
          <div style="margin-top:15px;font-size: 20px;font-weight: bold">集中器在线数：<span style="font-size:20px;color: #1cdb35">{{this.onlineTerm}}</span>
          </div>
<!--          <div style="margin-top:15px;font-size: 20px;font-weight: bold">区域集中器总数为：<span style="font-size:20px;color: #e39012">{{this.areaTotalTerm}}</span>-->
<!--          </div>-->
          <div style="margin-top:15px;font-size: 20px;font-weight: bold">区域集中器在线数：<span style="font-size:20px;color: #1cdb35">{{this.onlineTermArea}}</span>
          </div>
        </el-col>
        <el-col :span="20" style="margin-top: 20px;">
          <el-form label-width="auto" :model="termForm">
            <div style="font-size: 20px;font-weight: bold;margin-bottom: 20px;">电表信息</div>
            <el-row :gutter="20" style="margin-top: 5px">
              <el-col :span="8">
                <el-form-item label="尖电量:" prop="online">
                  <div size="small" >{{minuteData.realFee1}}</div>
                  <div size="small" style="display: none" >{{minuteData.realFee1}}kw/h</div>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="峰电量:" prop="address">
                  <div size="small" >{{minuteData.realFee2}}kw/h</div>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="平电量:" prop="installAddr">
                  <div size="small" >{{minuteData.realFee3}}kw/h</div>
                </el-form-item>
              </el-col>
            </el-row>
            <el-row :gutter="20">
              <el-col :span="8">
                <el-form-item label="谷电量:" prop="name">
                  <div size="small" >{{minuteData.realFee4}}kw/h</div>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="总电量:" prop="installDate">
                  <div size="small" >{{minuteData.realRealSumFlow}}kw/h</div>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="采集时间:" prop="protocolCode">
                  <div size="small" >{{minuteData.realCollTime}}</div>
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
        </el-col>
      </el-row>
      <el-row style="height: 8.2rem">
        <el-col :span="5" style="height: 8.2rem">
          <el-row :gutter="20" style="background:rgba(21,21,167,0.18);color:red;font-weight: bold ">
            <el-col :span="10">
              <div style="font-size: 16px;line-height:37px;">集 中 器</div>
            </el-col>
            <el-col :span="14">
              <el-col :span="24">
                <el-select size="medium" style="cursor:pointer;" v-model="areaId" placeholder="请选择组织区域" @change="selectChange" >
                  <el-option
                    v-for="(item,index) in allGrade"
                    :key="index"
                    :label="item.parentName"
                    :value="item.parentId">
                  </el-option>
                </el-select>
              </el-col>
            </el-col>
          </el-row>
          <!--            list列表展示集中器-->
          <el-row :gutter="20" style="height: 7.5rem;overflow-y: scroll">
            <loading v-if="loadVisible" :text="'Loading'"></loading>
            <el-card :id="i" v-for="(info,i) in infoList" @click.native="getMeter1(info, i)" class="type"
                     :key="i"  :class="{'bg_color':current_index===i}">
              <div class="content">
                <ol>
                  <li style="font-size: 10px;font-weight: 100">地 址:<span style="font-weight: normal;font-size: 10px">{{info.installAddr}}</span>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;
                    &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; &nbsp;&nbsp;&nbsp; 名 称:<span style="font-weight: normal;font-size: 10px">{{info.name}}</span>
                  </li>
                </ol>
              </div>
            </el-card>
          </el-row>
        </el-col>
        <el-col :span="18" style="margin-left: 0.5rem">
          <el-row :gutter="20">
            <div style="font-size: 20px;font-weight: bold;background:#f3f3f4;width: 100%;height: 40px;line-height: 40px;">
              <el-select size="small" v-model="meterType" placeholder="请选择表具类型" @change="getMeter2()">
                <el-option
                  v-for="(item, index) in allType"
                  :key="'new_'+index"
                  :label="item.name"
                  :value="item.value">
                </el-option>
              </el-select>
              <el-button style="float: right" size="medium" type="primary"
                         :disabled="multipleSelection.length === 0 || online "
                         >集 抄
              </el-button>
              <!--              <el-button style="float: right" size="medium" type="primary" @click="test()">测试</el-button>-->
            </div>
          </el-row>
          <el-row :gutter="20" style="margin-top: 20px">
            <!--            表数据表格-->
            <el-table size="small" :data="listData" highlight-current-row v-loading="loading" border ref="table" :header-cell-style="{background:'#02a7f0'}"
                      element-loading-text="正在加载中" style="width: 100%" :key="controKey" :height="tableHeight" :row-class-name="tableRowClassName">
              <el-table-column align="center" type="selection" width="50em">
              </el-table-column>
              <el-table-column v-if="false" sortable prop="id" label="有线水表ID">
              </el-table-column>
              <el-table-column type="index" align="center"  label="序号" :index="table_index" width="60">
              </el-table-column>
              <el-table-column prop="meterType" align="center"  :show-overflow-tooltip='true' label="类型" min-width="100em">
                <template slot-scope="scope">
<!--                  <div slot="reference" class="name-wrapper" v-if="scope.row.meterType == '0'">-->
<!--                    <el-tag size="medium" type="warning">冷水水表</el-tag>-->
<!--                  </div>-->
                  <div slot="reference" class="name-wrapper" v-if="scope.row.meterType == '4'">
                    <el-tag size="medium" type="danger">电表</el-tag>
                  </div>
                </template>
              </el-table-column>
              <el-table-column prop="meterAddress" align="center"  :show-overflow-tooltip='true' label="表地址" min-width="180em">
              </el-table-column>
              <el-table-column prop="termId" align="center"  :show-overflow-tooltip='true' label="集中器地址" min-width="150em">
              </el-table-column>
              <el-table-column prop="name" align="center"  :show-overflow-tooltip='true' label="表具名称" min-width="180em">
              </el-table-column>
<!--              <el-table-column sortable prop="realRead" align="center" v-if="meterType==='4884'"  :show-overflow-tooltip='true' label="读数(m³)" min-width="150em">-->
<!--              </el-table-column>-->
<!--              <el-table-column sortable prop="realRead" align="center" v-if="meterType==='0'" :show-overflow-tooltip='true' label="读数(kwh)" min-width="150em">-->
<!--              </el-table-column>-->
<!--              <el-table-column  prop="message"  align="center"  :show-overflow-tooltip='true' label="抄读信息" min-width="150em">-->
<!--              </el-table-column>-->
              <el-table-column align="center" label="操作" width="150em" :render-header="tableAction">
                <template slot-scope="scope">
                  <el-button size="mini" type="primary" @click="toReadMeter(scope.$index, scope.row)" circle  class="fa fa-copyright"></el-button>
                </template>
              </el-table-column>
            </el-table>

            <Pagination v-bind:pageSize="pageSize" v-bind:child-msg="pageparm" @callFather="callFather"
                        ref="pageRef"></Pagination>
          </el-row>
        </el-col>
      </el-row>


<!--    <el-dialog title="实时抄表" :closeOnClickModal="false" :visible.sync="visible" width="20%" @close="closeDialog"-->
<!--               class="upload-dialog">-->
<!--      <el-row>-->
<!--        <loading v-if="readVisible" class="loader"></loading>-->
<!--      </el-row>-->
<!--    </el-dialog>-->
  </div>
</template>

<script>
  import {getTermData, getMeter} from '../../api/termTest'
  import {getGradeToSel} from "../../api/fileMG"
  import Pagination from '../../components/Pagination'
  import loading from '../../components/loading'
  import loadingTwo from '../../components/loadingTwo'
  import store from "../../vuex/store";
  import HelpHint from '../../components/HelpHint'

  export default {
    name: "testTermOnline",
    components: {
      Pagination,
      loading,
      loadingTwo,
      HelpHint
    },
    data() {
      return {

        lockReconnect: false, //是否真正建立连接
        timeout: 58 * 1000, //58秒一次心跳
        timeoutObj: null, //心跳心跳倒计时
        serverTimeoutObj: null, //心跳倒计时
        timeoutnum: null, //断开 重连倒计时

        minuteData: {
          realFee1: "",
          realmeterAddress: "",
          realPn: "",
          realFee2: "",
          realFee3: "",
          realRealSumFlow: "",
          realCollTime: "",
        },
        socket:"",
        meterType:"4",
        allType:[
         {
            name:"电表",
            value:"4"
          }
        ],
        on:"在线",
        off:"离线",
        readVisible: false,
        termAddress:"",
        pn:"",
        infoList: [],
        current_index: -1,
        onlineTerm: "",
        areaTotalTerm:"",
        onlineTermArea:"",
        termForm: {
          assetNo: "",
          status: "",
          installAddr: "",
          name: "",
          protocolColde: "",
          installDate: ""
        },
        termSum: "",
        meterAddress: "",
        tableHeight: this.tableHeight,
        realRead: "--",

        listData: [],
        loading: false,
        visible: false,
        // editForm: {},
        allGrade: [],
        areaId: "",
        pageSize: [10, 20,30,40],
        pageparm: {
          currentPage: 1,
          pageSize: 10,
          total: 10
        },
        formInline: {
          page: 1,
          limit: 10,
        },
        multipleSelection: [],
        websock:null,
        loadVisible: false,
        controKey: 0,
        online: true,
        flag : false,
      }
    },
    /**
     * 创建完毕
     */
    created() {
      this.getAllGradeToSel()
      this.initWebSocket();
      let row = {
        code:"0001",
        object:{
         address : "03053333"
        },
        user: store.state.userId
      }

    },
    toRedirect(data){
      if(data.code == "0001" && data.token != this.$store.state.token){
        closeWebsocket()
        sessionStorage.removeItem("ORG_NAME")
        sessionStorage.removeItem("user")
        sessionStorage.setItem("userInfo",false)
        this.$store.state.tabRout =[{menuName:"首页",url:"/dataView/firstIndex",parentMenuId:"1"}]
        this.$message.warning({title:"提示",message:"已在其他地方登录本账号！", duration:5000})
        this.$router.push({path: `/login`})
      }
    },
    failGetOnlineChange(){
      //单点登录监听出现异常
      console.log("单点登录监听出现异常")
    },
    mounted() {

      this.tableHeight = window.innerHeight  - 400 + "px" ; //260即表格之外的固定高度
      window.onresize = () => {
        return (() => {
          this.tableHeight = window.innerHeight  - 400 + "px"
        })();
      };
    },
    //离开页面销毁程序
    destroyed() {
      this.websock.close(); //离开路由之后断开websocket连接
    },
    methods: {

      // e.data='"{"collTime":1722925440000,"fee1":1052.79,"fee2":1211.83,"fee3":1525.35,"fee4":340.20,"realSumFlow":4130.18}"';
      getMessage: function (msg) {
        console.log(msg.data)
        const redata = JSON.parse(msg.data);
        console.log(redata)
        console.log(redata.realSumFlow)
        this.minuteData.realRealSumFlow = redata.realSumFlow
        this.minuteData.realFee3 = redata.fee3
        this.minuteData.realFee2 = redata.fee2
        this.minuteData.realFee1 = redata.fee1
        this.minuteData.realFee4 = redata.fee4

      },

  //数据接收
  //  websocketonmessage(e) {
  //   console.log(JSON.parse(e.data))
  //    console.log(redata);
  //    const redata = JSON.parse(e.data);
  //    this.realRealSumFlow = redata.realSumFlow
  //    this.realFee3 = redata.realFee3
  //    this.realFee2 = redata.realFee2
  //    this.realFee1 = redata.realFee1
  //    this.realCollTime = redata.realCollTime
  //    this.websock.onclose = this.websocketclose;
  //    window.dispatchEvent(new CustomEvent('onmessageWS', {
  //     detail: {
  //       data: JSON.parse(e.data)
  //     }
  //   }))
  // },
      //表格提示
      tableAction(){
        return this.$createElement('HelpHint', {
          props: {
            content: '实时抄表'
          }
        }, '操作');
      },
      //表格序号
      table_index(index){
        return (this.pageparm.currentPage-1) * this.pageparm.pageSize + index + 1
      },
      //表格间隔行换行变换样式
      tableRowClassName({ row, rowIndex }) {
        if ((rowIndex + 1) % 2 === 0) {
          return "warning-row"; //类名
        } else {
          return "success-row"; //类名
        }
      },
      getOnlineChange(data) {
        //此0001为集中器在线集中器设备的数量信息
        if(data.code != "0001"){
          //实时抄表socket直接返回
          return
        }
        if (data.changeTerm == null || data.changeTerm == undefined) {
          //返回数据为空，直接返回
          return
        }
        let count = 0
        this.infoList.forEach(value => {
          if (value.address == data.changeTerm.address) {
            value.online = data.changeTerm.online
          }
          if(value.online == 1){
            count ++
          }
        })
        this.onlineTerm = count
      },

      selectChange() {
        this.listData = []
        this.infoList = []
        this.termSum = ""
        this.onlineTermArea = ""
        this.areaTotalTerm = ""
        this.onlineTerm = ""
        this.current_index = -1
        this.$forceUpdate()
        this.termForm = {}
        this.getTermData1()
        this.multipleSelection = []
      },
      getAllGradeToSel() {
        const that = this
        that.allGrade = []
        getGradeToSel({id: ""}).then((res) => {
          res.forEach(value => {
            let option = {
              parentId: '',
              parentName: ''
            }
            option.parentId = value.id
            option.parentName = value.areaName
            that.allGrade.push(option)
          })
          that.formInline.areaId = that.allGrade[0].parentId;
          that.getMeter2()
        }).catch(() => {
          return;
        })
      },
      //根据集中器ID获取表信息
      getMeter1(info, index) {
        //修改样式点击的el-card的
        const that = this
        that.current_index = index
        that.formInline.areaId = that.areaId
        that.formInline.address = info.address
        that.formInline.meterType = that.meterType
        that.termForm = info
        that.online = (info.online === 0)
        getMeter(that.formInline).then((res) => {
          that.listData = res.records
          if(that.listData != null){
            that.listData.forEach(value => {
              value.realRead = "--"
              value.message = ""
              value.assetNo = info.assetNo
            })
          }
          that.pageparm.currentPage = that.formInline.page
          that.pageparm.pageSize = that.formInline.limit
          that.pageparm.total = res.total
        })
      },
      //切换类型是查询所属表具
      getMeter2(){
        this.loading = true
        this.formInline.areaId = this.areaId
        this.formInline.meterType = this.meterType
        getMeter(this.formInline).then((res) => {
          this.loading = false
          this.listData = res.records
          if(this.listData != null){
            this.listData.forEach(value => {
              value.assetNo = this.formInline.address
            })
          }
          this.pageparm.currentPage = this.formInline.page
          this.pageparm.pageSize = this.formInline.limit
          this.pageparm.total = res.total
        })
      },
      //获取集中器信息
      getTermData1() {
        const that = this
        that.infoList = []
        if (that.areaId == "" || that.areaId == undefined) {
          this.$notify.warning("请选择组织区域")
          return;
        }
        that.loadVisible = true
        getTermData(that.areaId).then((res) => {
          if(res.code === 500){
            this.$message.error("系统请求出现错误，请联系开发人员。")
          }
          that.infoList = res.list
          that.termSum = res.totalTerm
          that.onlineTerm = res.onlineTerm
          that.onlineTermArea = res.onlineTermArea
          that.areaTotalTerm = res.areaTotalTerm
          that.loadVisible = false
        }).catch(res => {
          that.$message.error("集中器信息查询出现异常，查询失败！")
          that.loadVisible = false
          that.areaId = ""
        })
      },
      //集中器多表齐抄
      // batchToReadMeter(selection) {
      //   // let address = []
      //   // selection.forEach(value => {
      //   //   address.push(value.meterAddress)
      //   // })
      //
      //   let object = {
      //       termAddress: selection[0].termId,
      //       pn:selection[0].pn,
      //       userId:selection[0].userId,
      //       message:"发送实时抄表请求",
      //       meterAddress: value.meterAddress
      //   }
      //   row.object = JSON.stringify(object)
      //   if (selection[0].termId == null || selection[0].termId == "" || address == []) {
      //     this.$notify.error("抄表选择表存在参数问题！")
      //     return
      //   }
      //   this.visible = true
      //   this.readVisible = true
      //   this.multipleSelection = []
      //
      //   //打开单点登录校验websocket
      //   const url = "ws://124.71.103.96:9006/websocket/"+store.state.currentUser.id+"readMeter"
      //   connectWebsocket(
      //     url,
      //     // 发送
      //     JSON.stringify(row),
      //     // 成功拿到后台返回的数据的回调函数
      //     (data) => {
      //       this.toRedirect(JSON.parse(data))
      //     },
      //     // websocket连接失败的回调函数
      //     () => {
      //       this.failGetOnlineChange()
      //     }
      //   );
      // },


      currentTime() {
        setInterval(this.formatDate, 500);
      },
      initWebSocket() {
        //初始化weosocket

        const wsuri = "ws://124.71.103.96:8086/socketServer/"+store.state.currentUser.id;
        this.websock = new WebSocket(wsuri);
        // 客户端接收服务端数据时触发
        this.websock.onmessage = this.websocketonmessage;
        // 连接建立时触发
        this.websock.onopen = this.websocketonopen;
        // 通信发生错误时触发
        this.websock.onerror = this.websocketonerror;
        // 连接关闭时触发
        this.websock.onclose = this.websocketclose;
      },
      // 连接建立时触发
      websocketonopen() {
        //开启心跳
        this.start();
        this.websock.send("heartCheck");
        //连接建立之后执行send方法发送数据

      },
      // 通信发生错误时触发
      websocketonerror() {
        console.log("出现错误");
        this.reconnect();
      },
      // 客户端接收服务端数据时触发
      websocketonmessage(e) {
        //收到服务器信息，心跳重置
        this.reset();

        if("heartCheck"==e.data){

        }else {
          console.log("收到的实时抄表数据为："+e.data);
          this.getMessage(e)
          this.$notify.success({message:"抄表成功"})
        }
      },
      websocketsend(Data) {
        //数据发送
        this.websock.send(Data);
      },
      // 连接关闭时触发
      websocketclose(e) {
        //关闭
        console.log("断开连接", e);
        //重连
        this.reconnect();
      },
      reconnect() {
        //重新连接
        var that = this;
        if (that.lockReconnect) {
          return;
        }
        that.lockReconnect = true;
        //没连接上会一直重连，设置延迟避免请求过多
        that.timeoutnum && clearTimeout(that.timeoutnum);
        that.timeoutnum = setTimeout(function () {
          //新连接
          that.initWebSocket();
          that.lockReconnect = false;
        }, 5000);
      },
      reset() {
        //重置心跳
        var that = this;
        //清除时间
        clearTimeout(that.timeoutObj);
        clearTimeout(that.serverTimeoutObj);
        //重启心跳
        that.start();
      },
      start() {
        //开启心跳
        console.log("开启心跳");
        var self = this;
        self.timeoutObj && clearTimeout(self.timeoutObj);
        self.serverTimeoutObj && clearTimeout(self.serverTimeoutObj);
        // self.websock.send("heartCheck"); //这里可以自己跟后端约定
        self.timeoutObj = setTimeout(function () {
          self.websock.send("heartCheck"); //这里可以自己跟后端约定
          var state = self.websock.readyState;
          console.log(state)
          //这里发送一个心跳，后端收到后，返回一个心跳消息，
          if (self.websock.readyState == 1) {
            //如果连接正常

          } else {

            //否则重连
            self.reconnect();
          }
          self.serverTimeoutObj = setTimeout(function () {
            //超时关闭
            self.websock.close();
          }, self.timeout);
        }, self.timeout);
      },

      // 销毁定时器
      beforeDestroy() {
        if (this.formatDate) {
          clearInterval(this.formatDate); // 在Vue实例销毁前，清除时间定时器
        }
      },


      //抄表
      toReadMeter(index, row) {
        this.meterAddress = row.meterAddress;
        this.realPn = row.pn;
        console.log(this.realmeterAddress);
        this.minuteData.realmeterAddress =row.meterAddress;
        let actions = {"termAddress":row.termId,"pn":row.pn,"userId":store.state.userId};
        // let actions = {"room":"007854ce7b93476487c7ca8826d17eba","info":"1121212"};
        this.websocketsend(JSON.stringify(actions));
        const now = new Date();
        const year = now.getFullYear();
        const month = now.getMonth() + 1;
        const day = now.getDate();
        const hour = now.getHours();
        const minute = now.getMinutes();
        const second = now.getSeconds();
        var formattedTime = year + "-" +
          (month < 10 ? "0" : "") + month + "-" +
          (day < 10 ? "0" : "") + day + " " +
          (hour < 10 ? "0" : "") + hour+ ":" +
          (minute < 10 ? "0" : "") + minute + ":" +
          (second < 10 ? "0" : "") + second;
        this.minuteData.realCollTime = formattedTime;
        // this.user = JSON.parse(localStorage.getItem('userdata'))
        // this.ORG_ACCOUNT = localStorage.getItem("ORG_ACCOUNT")
        this.index = "0"
        this.$nextTick(()=>{
          this.activeIndex = parseInt(this.index)
        })

        // this.websocketsend(JSON.stringify(actions));
        const that = this
        // that.editForm.meterType = row.meterType
        // that.editForm.meterAddress = row.meterAddress
        // that.editForm.instLoc = row.instLoc
        // that.editForm.assetNo = row.termId
        // that.realRead = "--"
        // if (row.termId == null || row.termId == "" || row.meterAddress == null || row.meterAddress == "") {
        //   that.$notify.error({title: "抄写失败", message: "抄表选择表存在参数问题！"})
        //   return
        // }
        // if (this.online) {
        //   that.$notify.error({title: "抄写失败", message: "集中器未在线！"})
        //   return
        // }

        let arr = []
        arr.push(row)
        that.readVisible = false
        that.visible = true

      },
      getMeterData(data) {
        const that = this

        that.listData.forEach(value => {
          data.meterDataList.forEach(res => {
            if (value.meterAddress === res.meterAddress) {
              value.realRead = res.meterData
              that.realRead = res.meterData //已弃用，弹窗中的显示读数
              value.message = "抄表成功"
            }
          })
        })
        that.$notify.success({message:"抄表成功"})
        that.controKey++
        if(data.isEnd == 1){
          this.closeDialog()
        }
      },
      callFather(parm) {
        this.formInline.page = parm.currentPage
        this.formInline.limit = parm.pageSize
        this.getMeter2()
      },
      closeDialog() {
        this.readVisible = false
        this.visible = false
      },
    }
  }

</script>

<style scoped>
  /deep/ .loader {
    margin-top: 70px;
    margin-bottom: 27px;
  }

  .readMeter {
    line-height: 16px;
    font-size: 16px;
    font-weight: normal
  }
  /deep/ .el-card.is-always-shadow, .el-card.is-hover-shadow:focus, .el-card.is-hover-shadow:hover {
    -webkit-box-shadow: 0 2px 12px 0 rgba(0,0,0,.1);
    box-shadow: 0 2px 12px 0 rgba(0,0,0,-1.9);
  }

  .type {
    width: 330px;
    margin-top: 10px;
    background: #fff;
    height: 95px;
    font-size: 16px;
    line-height: 20px;
    cursor: pointer;
    border-radius: 5px
  }

  .bg_color {
    background: linear-gradient(135deg, #2AFADF 10%, #4C83FF 100%) !important;
  }

  .upload-dialog .el-dialog__header {
    border-bottom: 1px solid #e8eaec;
    background: #b9d3f0;
  }

  .box-card1 {
    float: left;
    border-right: 1px solid grey;
    width: 20%;
    height: 800px;
  }

  .box-card2 {
    margin-left: 2%;
    float: left;
    width: 78%;
    height: 800px;
  }

  .box-card {
    margin-top: 15px;
    width: 100%;
    height: 800px;
  }

</style>
