<template>
  <div class="fun_wrap" :style="{background: '#f0f3f4', marginLeft: '0px'}">
      <el-container v-bind:style="{height: wrapHeight + 'px',position: 'relative'}">
          <el-aside v-show="carTreeIsShow" id="orgTreeBox" v-bind:style="{width: '230px', marginTop: '-20px', boxShadow: '1px -2px 3px 0px #dcdfe6', background: '#f9f9f9'}">
            <el-tabs id="carMonitor_tabs" v-model="carTreeTabSelect" v-bind:style="{marginTop: '10px'}">
                <el-tab-pane label="所有车辆" name="first">
                    <div id="carMonitor_cartree_searinput" style="margin: 25px 0 15px 0;">
                        <el-autocomplete size="mini" clearable placeholder="请输入内容" v-model="carTreeSearInput" class="input-with-select" :fetch-suggestions="carTreeComplteInputSear" @select="carTreeDataSelect" >
                            <el-select v-model="carTreeSearSelect" @change="carTreeSearSelectEvent" slot="prepend">
                                <el-option label="车牌号" value="carNo"></el-option>
                                <el-option label="SIM卡号" value="actualSimNo"></el-option>
                                <el-option label="终端号" value="tboxId"></el-option>
                                <el-option label="VIN码" value="carVin"></el-option>
                            </el-select>
                            <el-button slot="append" @click="carTreeSearIconEvent" icon="el-icon-search"></el-button>
                        </el-autocomplete>
                        <span @click="showTreeConfig" :style="{fontSize: '16px', cursor: 'pointer', verticalAlign: 'middle', marginLeft: '5px'}" class="el-icon-setting"></span>
                    </div>
                    <EasyScrollbar>
                        <div>
                            <ul class="ztree" v-bind:style="{display: 'table'}" id="car_monitor_tree"></ul>
                        </div>
                    </EasyScrollbar>
                </el-tab-pane>
                <el-tab-pane label="关注车辆" name="second">
                    <group-config></group-config>
                </el-tab-pane>
            </el-tabs>
          </el-aside>
          <div v-show="!carTreeIsShow" @click="openCarTree" class="car_moni_tree_open">
             <img src="static/img/orgtreeshow_icon.png" alt="">
          </div>
          <el-container v-bind:style="{minWidth: '850px'}" class="car_monitor_right">
              <el-main ref="hqMapDivRef">
                  <!-- AMap -->
                  <hq-map :mapStyle="mapStyle" :zoomNum="13"></hq-map>
              </el-main>
              <el-footer :style="{height: 'auto', position: 'relative'}">
                  <div :style="{position: 'absolute', top: '0', left: '50%', marginLeft: '-74px', cursor: 'pointer'}">
                      <img @click="hideCarTable" src="static/img/carMonitor/table_down.png" alt="">
                      <img @click="openCarTable" src="static/img/carMonitor/table_up.png" alt="">
                  </div>
                  <el-tabs v-show="isCarTableShow" class="myTabs_wrap" v-model="tabSelect" :style="{marginTop: '10px'}">
                    <el-tab-pane label="传统车" name="normal">
                        <el-table ref="car_moni_nortable" :height="tabTableHeight" tooltip-effect="dark" :stripe="true" size="mini" highlight-current-row :border="true">
                            <el-table-column :show-overflow-tooltip="showTipFlg" width="34" header-align="left" type="selection" label=""></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" width="52" header-align="left" type="index" label="序号"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="carVin" label="VIN码"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="productionDate" label="生产日期"> </el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="tboxId" label="终端号"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="actualSimNo" label="SIM卡号"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="iccid" label="ICCID"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="createTime" label="创建时间"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="createUserName" label="创建者"></el-table-column>
                        </el-table>
                        <el-row :span="24">
                            <el-col :span="20" :offset="2" :style="{textAlign: 'center'}">
                                <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="[15, 20, 30, 50]" :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper" background :total="TotalPages">
                                </el-pagination>
                            </el-col>
                        </el-row>
                    </el-tab-pane>
                    <el-tab-pane label="新能源车" name="energy">
                        <el-table ref="car_moni_enertable" :height="tabTableHeight" tooltip-effect="dark" :stripe="true" size="mini" highlight-current-row :border="true">
                            <el-table-column :show-overflow-tooltip="showTipFlg" width="34" header-align="left" type="selection" label=""></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" width="52" header-align="left" type="index" label="序号"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="carVin" label="VIN码"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="productionDate" label="生产日期"> </el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="tboxId" label="终端号"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="actualSimNo" label="SIM卡号"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="iccid" label="ICCID"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="createTime" label="创建时间"></el-table-column>
                            <el-table-column :show-overflow-tooltip="showTipFlg" header-align="left" prop="createUserName" label="创建者"></el-table-column>
                        </el-table>
                        <el-row :span="24">
                            <el-col :span="20" :offset="2" :style="{textAlign: 'center'}">
                                <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="[15, 20, 30, 50]" :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper" background :total="TotalPages">
                                </el-pagination>
                            </el-col>
                        </el-row>
                    </el-tab-pane>

                  </el-tabs>

              </el-footer>
              <div v-show="carTreeIsShow" @click="hideCarTree" class="car_moni_tree_close">
                 <img src="static/img/orgtreehide_icon.png" alt="">
              </div>
          </el-container>

          <!-- 树配置 -->
          <tree-config @loadTreeByConfig="loadTreeByConfig" @resetTreeConfigVsible="resetTreeConfigVsible" :treeConfigVisible="treeConfigVisible" ></tree-config>

      </el-container>

  </div>
</template>
<script>
import axios from 'axios'
// import router from 'router/index'
import treeConfig from './treeConfig'
import groupConfig from './groupConfig'
import hqMap from 'components/HQMap'
import api from 'api/carMonitor'
export default {
    components: {
        // 车辆树配置
        treeConfig,
        // 地图部分
        hqMap,
        // 分组
        groupConfig
    },

    data(){
        return {

            // 车辆树配置属性 start
            treeConfigVisible: false,
            // 车辆树配置属性 end
            // 车辆树属性 start
            numStuTime: 5000,
            treeType: 1,          //1车辆树； 2 关注车辆树 3 属性树
            // 车辆树属性 end
            mapStyle: 'height: 602px',
            showTipFlg: true,
            form: {},

            wrapHeight: 802,
            carTreeTabSelect: 'first',
            tabSelect: 'normal',
            tabTableHeight: 200,
            carTreeIsShow: true,
            autoCarTreeStaTimer: null,
            currentPage: 1,
            pageSize: 15,
            TotalPages: 100,
            isCarTableShow: true,
            carTreeSearInput: '',
            carTreeSearSelect: 'carNo',
            carTreeExpandPathArr: [],
            carTreeSetting: {
                check: {
                    enable: true,
                    chkStyle: "checkbox"
                },
                async: {
                    enable: true,
                    url: axios.defaults.baseURL + '/tree/userOrgTree',
                    type: "get",
                    autoParam: ["id=parentNodeId"]
                },
                callback: {
                    onAsyncSuccess: this.carTreeOnSuccesEvent,
                    onExpand: this.carTreeExpand
                }
            },
            confCarTreeSetting:{
                check: {
                    enable: true,
                    chkStyle: "checkbox"
                },
                async: {
                    enable: true,
                    url: axios.defaults.baseURL + '/tree/carset/tree',
                    type: "get",
                    // autoParam: ["id=nodeId", "pId=pNodeId"]
                    autoParam: ["id=nodeId", "pId=pNodeId"]
                    // otherParam: ["id=nodeId", "pId=pNodeId"]
                },
                callback: {
                    onAsyncSuccess: this.carTreeOnSuccesEvent,
                    onExpand: this.carTreeExpand
                }
            },
            TreeRootData: [{
                "checked":"false",
                "children":[],
                "icon":"",
                "iconClose":"",
                "iconOpen":"",
                "id":"-1",
                "pId": 'null',
                "tId":"-1",
                "isParent":"true",
                "name":"车辆监控中心",
                "open":"false",
                "parentTId":"0",
                "type":0
            }]

        }
    },
    created() {
        this.wrapHeight = this.$store.state.home.settings.deviceHeight - 22;
    },
    mounted() {
        var carTree = $.fn.zTree.init($("#car_monitor_tree"), this.carTreeSetting, this.TreeRootData);
        var treeNodeS = carTree.getNodes()

        carTree.expandNode(treeNodeS[0], true)
        $('#carMonitor_tabs .easy-scrollbar__wrap').height(this.wrapHeight - 65)

        this.autoFreshCartreeStaAndNum()

        this.getCartreeStateAndNum()

    },
    activated(){
        var isFreshCarTree = this.$store.state.home.isFreshCarTree

        if(isFreshCarTree){ //全局车辆树更新
            var carTree = $.fn.zTree.init($("#car_monitor_tree"), this.carTreeSetting, this.TreeRootData);
            var treeNodeS = carTree.getNodes()

            carTree.expandNode(treeNodeS[0], true)

            this.$store.state.home.isFreshCarTree = false
        }
    },
    watch: {
        isCarTableShow(newVal, oldVal) {
            let h = 500

            if(newVal && newVal != oldVal) {
                h = this.wrapHeight - this.tabTableHeight
            } else {
                h = this.wrapHeight
            }
            this.resetMapStyle(h)
        }
    },
    methods: {
        getTableData(){
            let data = {
                pageNum: this.currentPage,
                pageSize: this.pageSize
            }

            console.log(data)
        },
        handleCurrentChange(val) {
            this.currentPage = val
            this.getTableData()
        },
        handleSizeChange(val) {
            this.pageSize = val
            this.getTableData()
        },
        showGroupConfig() {
            this.groupConfigVisible = true
        },
        showTreeConfig() {
            this.treeConfigVisible = true
        },
        resetTreeConfigVsible(val) {
            this.treeConfigVisible = val
        },
        resetMapStyle(h) {
            this.mapStyle = 'height: ' + h + 'px'
        },
        hideCarTable(){
            this.isCarTableShow = false
        },
        openCarTable(){
            this.isCarTableShow = true
        },
        hideCarTree(){
            this.carTreeIsShow = false
        },
        openCarTree(){
            this.carTreeIsShow = true
        },
        // *************************************************************************车辆树逻辑*****************************************************

        carTreeOnSuccesEvent(event, treeId, treeNode){  //车辆树异步加载成功回调
            var cartree = $.fn.zTree.getZTreeObj("car_monitor_tree")
            var childArr = []

            childArr = treeNode.children

            if(childArr){
                this.getCartreeStateAndNumData(childArr)
                this.changeCartreeShowSelect(childArr)
            }

            if(childArr.length > 0){
                for(var item of childArr){
                    /*   车辆树查询开始      */
                    var nodeid = item.id
                    var arrLeng = this.carTreeExpandPathArr.length  //车辆树查询 树节点path 集合
                    var index = this.carTreeExpandPathArr.indexOf(nodeid)
                    // var targetNode = this.carTreeExpandPathArr[arrLeng - 1]

                    if(index != -1){
                        if(index < arrLeng - 1){
                            var nextnode = cartree.getNodeByParam("id", this.carTreeExpandPathArr[index], null)

                            if(nextnode){
                                cartree.expandNode(nextnode, true)
                            }
                        }else if(index == arrLeng - 1){
                            var nextnode1 = cartree.getNodeByParam("id", this.carTreeExpandPathArr[index], null)

                            cartree.cancelSelectedNode()

                            cartree.selectNode(nextnode1)
                            cartree.checkNode(nextnode1, true, true, true)
                        }
                    }

                    /*   车辆树查询结束      */
                }
            }

        },
         // *************************************************************************车辆树数量状态*************************************************
        autoFreshCartreeStaAndNum(){
            var that = this

            if(this.autoCarTreeStaTimer){
                clearInterval(this.autoCarTreeStaTimer)
            }
            this.autoCarTreeStaTimer = window.setInterval(function(){
                that.getCartreeStateAndNum()

            }, this.numStuTime)

            // this.$store.commit('addTimer', this.autoCarTreeStaTimer);

            this.GLOBAL.timerArr.push(this.autoCarTreeStaTimer)

        },
        carTreeExpand(){

        },

        getCartreeStateAndNum(){
            var cartree = $.fn.zTree.getZTreeObj("car_monitor_tree")
            var nodes = cartree.getNodes()

            nodes = cartree.transformToArray(nodes)
            this.getCartreeStateAndNumData(nodes)
        },
        getCartreeStateAndNumData(nodes){
            var cartree = $.fn.zTree.getZTreeObj("car_monitor_tree")
            var resArr = []

            if(this.treeType == 3){
                if(nodes.length > 0){
                    for(var item3 of nodes){
                        if(item3.isParent){
                            resArr.push(item3.id)
                        }
                    }
                }
            }else{
                if(nodes.length > 0){
                    for(var item of nodes){
                        if(item.isParent){
                            resArr.push(item.id)
                        }
                    }
                }
            }

            var obj = {
                nodeIds: JSON.stringify(resArr),
                treeType: this.treeType
            }

            api.getCarTreeStateAndNum(obj).then((response) =>{
                var orgnumArr =  response.data.orgNums
                var carStatus =  response.data.carStatus

                if(response.data.orgNumUpdate){  //数量是否需要更新
                    if(orgnumArr instanceof Array){  //遍历组织数量数组，组织节点数量
                        for(var itemnum of orgnumArr){
                            var itemnumspli = itemnum.split("|")
                            var nodeid = itemnumspli[0]
                            var itemnumStr = itemnumspli[1]
                            var itemnumArr = itemnumStr.split("_")

                            var carTatNum = itemnumArr[0]    //总车辆数
                            var carOffNum = itemnumArr[1]    //离线车辆数
                            var carUnknNum = itemnumArr[2]    //未知车辆数

                            var carOnlineNum = carTatNum - carOffNum - carUnknNum
                            var treeNode = cartree.getNodeByParam('id', nodeid, null)

                            if(treeNode){
                                var baseName = treeNode.name.split("[")[0]

                                treeNode.name = baseName + "[" + carOnlineNum + " / " + carTatNum + "]"
                                cartree.updateNode(treeNode)
                            }
                        }
                    }
                }
                if(carStatus instanceof Array){  //遍历车辆节点状态
                    for(var itemSta of carStatus){
                        var nodeResArr = itemSta.split("|")
                        var nodeResId = nodeResArr[0]
                        var nodeResSta = nodeResArr[1]  //null未知；6未知；1行驶在线；2报警行驶在线；3停车在线；4报警停车在线；5离线
                        var carNode = cartree.getNodeByParam('id', nodeResId, null)

                        if(nodeResSta == "null"){
                            nodeResSta = 6
                        }

                        if(carNode){
                            carNode.iconSkin = 'cartree_carnode_' + nodeResSta
                            cartree.updateNode(carNode)
                        }

                    }
                }

                var that = this

                if(response.data.orgNumExist){
                    if(this.autoCarTreeStaTimer){
                        clearInterval(this.autoCarTreeStaTimer)
                    }
                    this.numStuTime = 20000  //更改定时器时间
                    this.autoCarTreeStaTimer = window.setInterval(function(){
                        that.getCartreeStateAndNum()
                    }, this.numStuTime)
                }else{
                    if(this.autoCarTreeStaTimer){
                        clearInterval(this.autoCarTreeStaTimer)
                    }
                    this.numStuTime = 2000  //更改定时器时间
                    this.autoCarTreeStaTimer = window.setInterval(function(){
                        that.getCartreeStateAndNum()
                    }, this.numStuTime)
                }

            }).catch(err =>{
                this.$message({
                    type: 'error',
                    duration: 1500,
                    showClose: true,
                    message: err.message
                });
            })
        },
        // *************************************************************************车辆树搜索**************************************************************
        carTreeSearSelectEvent(){
            $('#carMonitor_cartree_searinput>div>.el-input__inner').focus()
            var cartree = $.fn.zTree.getZTreeObj("car_monitor_tree")
            var treenodes = cartree.transformToArray(cartree.getNodes())

            this.changeCartreeShowSelect(treenodes)
        },
        changeCartreeShowSelect(nodesArr){  //根据车辆树搜索下拉框车辆树显示对应Type
            var cartree = $.fn.zTree.getZTreeObj("car_monitor_tree")
            var nodetype = 'carNo'

            if(this.carTreeSearSelect == 'actualSimNo'){
                nodetype = 'sim'
            }else if(this.carTreeSearSelect == 'tboxId'){
                nodetype = 'tboxId'
            }else if(this.carTreeSearSelect == 'carNo'){
                nodetype = 'carNo'
            }else{
                nodetype = 'vin'
            }

            for(var node of nodesArr){
                if(!node.isParent){
                    if(nodetype != 'carNo'){
                        node.name = node.carNo + '(' + node[nodetype] + ')'
                    }else{
                        node.name = node.carNo
                    }
                    cartree.updateNode(node)
                }
            }
        },
        carTreeSearIconEvent(){
            $('#carMonitor_cartree_searinput>div>.el-input__inner').focus()
        },
        carTreeDataSelect(cbdata){  //车辆树搜索弹框选中勾选树节点
            var cartree = $.fn.zTree.getZTreeObj("car_monitor_tree")

            return this.getCarTreeNodePath(cbdata).then((dataStr) => {

                var pathArr = []

                if(dataStr){
                    pathArr = dataStr.split("_")

                    this.carTreeExpandPathArr = pathArr
                    var carNodeid = pathArr[pathArr.length - 1]

                    for(var item of pathArr){
                        var node = cartree.getNodeByParam("id", item, null)

                        if(node){
                            cartree.expandNode(node, true)
                            if(node.id == carNodeid){
                                cartree.selectNode(node)
                                cartree.checkNode(node, true, true, true)
                            }
                        }
                    }
                }

            })
        },
        getCarTreeNodePath(data){
            var resStr = ""
            var obj = {
                carUuid: data.carUuid,
                treeType: this.treeType
            }

            return api.carTreeNodePath(obj).then((response) =>{
                resStr = response.data
                return resStr
            }).catch(err =>{
                this.$message({
                    type: 'error',
                    duration: 1500,
                    showClose: true,
                    message: err.message
                });
            })

        },
        carTreeComplteInputSear(queryString, cb){               //车辆树输入框远程搜索
            var obj = {}

            if(this.carTreeSearSelect == "carNo"){
                obj = {
                    field: 'carNo',
                    carNo: queryString,
                    treeType: this.treeType
                }
            }else if(this.carTreeSearSelect == "carVin"){
                obj = {
                    field: 'carVin',
                    carVin: queryString,
                    treeType: this.treeType
                }
            }else if(this.carTreeSearSelect == "tboxId"){
                obj = {
                    field: 'tboxId',
                    tboxId: queryString,
                    treeType: this.treeType
                }
            }else if(this.carTreeSearSelect == 'actualSimNo'){
                obj = {
                    field: 'actualSimNo',
                    actualSimNo: queryString,
                    treeType: this.treeType
                }
            }

            api.carTreeSearInput(obj).then((response) =>{
                if(response.data){
                    if(this.carTreeSearSelect == "carNo"){
                        for(let i of response.data){
                            if(i.carNo){
                                i.value = i.carNo
                            }
                        }
                    }else if(this.carTreeSearSelect == "carVin"){
                        for(let i of response.data){
                            if(i.carVin){
                                i.value = i.carVin
                            }
                        }
                    }else if(this.carTreeSearSelect == "tboxId"){
                        for(let i of response.data){
                            if(i.carNo){
                                i.value = i.tboxId
                            }
                        }
                    }else if(this.carTreeSearSelect == 'actualSimNo'){
                        for(let i of response.data){
                            if(i.actualSimNo){
                                i.value = i.actualSimNo
                            }
                        }
                    }
                    return cb(response.data)
                }else{
                    return cb([])
                }
            }).catch(err =>{
                this.$message({
                    type: 'error',
                    duration: 1500,
                    showClose: true,
                    message: err.message
                });
            })
        },

        //********************************************************************车辆树显示属性配置********************************************** */
        loadTreeByConfig(Flg) {
            var carTree = null

            if(Flg){
                this.treeType = 3
                carTree = $.fn.zTree.init($("#car_monitor_tree"), this.confCarTreeSetting, this.TreeRootData);
            }else{
                carTree = $.fn.zTree.init($("#car_monitor_tree"), this.carTreeSetting, this.TreeRootData);
                this.treeType = 1
            }

            var treeNodeS = carTree.getNodes()

            carTree.expandNode(treeNodeS[0], true)
        }


    }
}
</script>
<style scoped>
.car_monitor_right{
    margin-left: 20px;
    position: relative;
    /* padding: 16px 35px 0px 20px; */
    background: #ffffff;
}
.car_moni_tree_open{
    position: absolute;
    left: -2px;
    top: 50%;
    margin-top: -39px;
    cursor: pointer;
}
.car_moni_tree_close{
    position: absolute;
    left: -11px;
    top: 50%;
    margin-top: -39px;
    cursor: pointer;
}
.hq-map-style {
        height: 500px;
}

</style>




