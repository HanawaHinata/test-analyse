<template>
    <a-spin :spinning="pageIsLoading">
        <div id="app">
            <div class="left_input_box">
                <div style="font-size: 12pt;margin: 0 0 10px 0;font-weight: bold;">字段设置</div>
                <table>
                    <tr>
                        <th>指标</th>
                        <th style="text-align: center;">低极致</th>
                        <th style="text-align: center;">高极致</th>
                        <th style="text-align: center;">权重</th>
                        <th style="text-align: center;">图形</th>
                        <th style="text-align: center;">操作</th>
                    </tr>
                    <template v-for="(item, index) in weightNumberList">
                        <tr :key="index">
                            <td>{{ item.name }}</td>
                            <td style="text-align: center;">
                                <input style="width: 50px;" v-model="weightNumberObject[item.name].minNumber"
                                       placeholder="0"/>
                            </td>
                            <td style="text-align: center;">
                                <input style="width: 50px;" v-model="weightNumberObject[item.name].maxNumber"
                                       placeholder="0"/>
                            </td>
                            <td style="text-align: center;">
                                <input style="width: 50px;" v-model="weightNumberObject[item.name].weightNumber"
                                       placeholder="0"/>
                            </td>
                            <td style="text-align: center;">
                                <a-checkbox :checked="isCheckedColumn(item)"
                                            @change="columnCheckedChange($event, item)"
                                            title="是否在ECharts中显示此列数据"/>
                            </td>
                            <td style="text-align: center;">
                                <a title="编辑" style="margin-right: 5px;" @click="editColumnItem(item)">
                                    <a-icon type="form"/>
                                </a>
                                <a-popconfirm title="确定要删除此列吗？" @confirm="deleteColumnItem(item)">
                                    <a title="删除" style="color: red">
                                        <a-icon type="delete"/>
                                    </a>
                                </a-popconfirm>
                            </td>
                        </tr>
                    </template>
                </table>


                <a-icon type="plus" style="margin: 10px 10px 20px auto;"
                        @click="editColumnItem({name: '新建字段',maxNumber: 20, minNumber: 0, isShow: true, weightNumber: 0, tbType: 'line'})"/>
                <a-button type="primary" icon="save" @click="saveColumnConfig"
                          style="margin-top: 10px;">保存字段配置
                </a-button>

                <div style="font-size: 12pt;margin: 40px 0 0 0;font-weight: bold;">总和图表设置</div>
                <div class="form_box">
                    <div class="item">
                        <div class="label">图表类型</div>
                        <a-radio-group class="input" v-model="currentChartType" button-style="solid">
                            <a-radio-button value="line">折线图</a-radio-button>
                            <a-radio-button value="bar">柱状图</a-radio-button>
                        </a-radio-group>
                    </div>
                    <div class="item">
                        <div class="label">是否堆叠</div>
                        <a-switch v-model="isStack"/>
                    </div>
                    <div class="item">
                        <div class="label">图形展示位置</div>
                        <a-radio-group class="input" v-model="xAxisPosition" button-style="solid">
                            <a-radio-button value="top">上</a-radio-button>
                            <a-radio-button value="bottom">下</a-radio-button>
                        </a-radio-group>
                    </div>
                </div>
            </div>
            <div class="right_display_box">
                <div class="table_box">
                    <a-table size="middle" :columns="chartsColumns" rowKey="id" :dataSource="tableDataSource" bordered
                             :pagination="false">
                        <template slot="riseStop" slot-scope="text">
                            {{ text==null||text===''?0:text }}
                        </template>
                        <template slot="time" slot-scope="text, record">
                            <a-checkbox :checked="isCheckedDataSource(record)"
                                        @change="dataSourceCheckedChange($event, record)"
                                        title="是否在ECharts中显示行数据"/>
                            {{ text }}
                        </template>
                        <template slot="action" slot-scope="text, records, index">
                            <a @click="editItem(records)">
                                <a-icon type="form"/>
                                编辑</a>
                            <a-divider type="vertical" :key="'record'+index" :text="text"/>
                            <a-popconfirm title="你要删除这行记录吗？" @confirm="deleteItem(records)">
                                <a style="color: #ff0000">
                                    <a-icon type="delete"/>
                                    删除</a>
                            </a-popconfirm>
                        </template>
                    </a-table>
                    <a-button style="display: block; margin: 20px auto;" @click="addListItem"
                              type="primary" icon="plus">添加
                    </a-button>
                </div>
                <div class="chart_box" id="chartBox">
                    <item-charts class="charts_item" title="情绪指数" index="endSum"
                                 :x-data="allDate" :y-data="getDataListByKey('endSum')"/>
                    <item-charts class="charts_item" title="情绪温度" index="dataSum"
                                 :x-data="allDate" :y-data="getDataListByKey('dataSum')"/>
                    <template v-for="(item, index) in tableColumnListFiltered">
                        {{getChartsType(item)}}
                        <item-charts :key="index" :index="index" class="charts_item" :title="item"
                                     :type="getChartsType(item)"
                                     :x-data="allDate" :y-data="getDataListByKey(item)"/>
                    </template>
                    <mixin-charts class="charts_item" title="总合数据图" index="dataAll" :type="currentChartType"
                                  :xAxisPosition="xAxisPosition" :x-data="allDate" :series="allDataSeries"/>

                </div>
            </div>

            <a-modal :title="(currentEditItem.id?'编辑':'新建')+'行数据'" :visible="showEditDialog" :width="800"
                     @ok="saveEditItem" @cancel="closeEditDialog">
                <a-form-model ref="ruleForm" :model="currentEditItem" :rules="rules"
                              :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }">
                    <a-form-model-item label="总和" prop="summation" style="margin: 0">
                        <a-input-number style="width: 100%;" v-model="currentEditItem.summation"/>
                    </a-form-model-item>
                    <a-form-model-item label="总和标准" prop="sumStandard" style="margin: 0">
                        <a-input-number style="width: 100%;" v-model="currentEditItem.sumStandard"/>
                    </a-form-model-item>
                    <a-form-model-item label="时间" prop="gpTime" style="margin: 0">
                        <a-input v-model="currentEditItem.gpTime"/>
                    </a-form-model-item>

                    <template v-for="(item, index) in currentEditItem.gpData">

                        <a-form-model-item :key="index" :label="item.name" style="margin: 0">
                            <a-input v-model="item.riseStop" placeholder="0"/>
                        </a-form-model-item>

                        <!--                        <div class="form_item_box" :key="index">-->
                        <!--                            <div class="label">-->
                        <!--                                <a-input placeholder="字段名" style="width: 100px;margin-right: 5px;" v-model="item.name"/>-->
                        <!--                                ：-->
                        <!--                            </div>-->
                        <!--                            <a-input-number class="input" v-model="item.riseStop"/>-->

                        <!--                            <a-button type="danger" icon="delete" @click="deleteFieldItem(index)"-->
                        <!--                                      style="margin-left: 10px;">-->
                        <!--                                删除-->
                        <!--                            </a-button>-->
                        <!--                        </div>-->
                    </template>
                    <!--                    <a-button type="primary" icon="plus" @click="addFieldItem"-->
                    <!--                              style="display: block;width: fit-content; margin: 0 0 0 auto;">添加字段项-->
                    <!--                    </a-button>-->


                </a-form-model>
            </a-modal>


            <a-modal :title="(currentColumnEditItem.id?'编辑':'新建')+'字段'" :visible="showColumnEditDialog" :width="500"
                     @ok="saveColumnEditItem" @cancel="closeColumnEditDialog">
                <a-form-model ref="columnForm" :model="currentColumnEditItem" :rules="columnRules"
                              :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }">
                    <a-form-model-item label="字段名称" prop="name" style="margin: 0">
                        <a-input style="width: 100%;" v-model="currentColumnEditItem.name"/>
                    </a-form-model-item>
                    <a-form-model-item label="低极致" prop="minNumber" style="margin: 0">
                        <a-input-number style="width: 100%;" v-model="currentColumnEditItem.minNumber"/>
                    </a-form-model-item>
                    <a-form-model-item label="高极致" prop="maxNumber" style="margin: 0">
                        <a-input-number style="width: 100%;" v-model="currentColumnEditItem.maxNumber"/>
                    </a-form-model-item>
                    <a-form-model-item label="权重" prop="weightNumber" style="margin: 0">
                        <a-input v-model="currentColumnEditItem.weightNumber"/>
                    </a-form-model-item>

                    <a-form-model-item label="是否展示图形" prop="isStack" style="margin: 0">
                        <a-switch :checked="isCheckedColumn(currentColumnEditItem)"
                                  @change="columnCheckedChange($event, currentColumnEditItem)"/>
                    </a-form-model-item>

                    <a-form-model-item label="图表类型" prop="gpTime" style="margin: 0">
                        <a-radio-group v-model="currentColumnEditItem.tbType" button-style="solid">
                            <a-radio-button value="line">折线图</a-radio-button>
                            <a-radio-button value="bar">柱状图</a-radio-button>
                        </a-radio-group>
                    </a-form-model-item>
<!--                    <a-form-model-item label="是否堆叠" prop="isStack" style="margin: 0">-->
<!--                        <a-switch v-model="currentColumnEditItem.tbPlace"/>-->
<!--                    </a-form-model-item>-->
<!--                    <a-form-model-item label="图形展示位置" prop="xAxisPosition" style="margin: 0">-->
<!--                        <a-radio-group v-model="currentColumnEditItem.tbOverlay" button-style="solid">-->
<!--                            <a-radio-button value="top">上</a-radio-button>-->
<!--                            <a-radio-button value="bottom">下</a-radio-button>-->
<!--                        </a-radio-group>-->
<!--                    </a-form-model-item>-->

                </a-form-model>
            </a-modal>

        </div>
    </a-spin>
</template>

<script>
// let mapChart = null;
// 引入基本模板
import itemCharts from "./components/itemCharts";
import MixinCharts from "./components/mixinCharts";


export default {
    name: 'App',
    components: {MixinCharts, itemCharts},
    data() {
        return {
            /** 基础 */
            baseUrl: "//139.224.41.178:8072/wealth",
            pageIsLoading: true,
            pageLoadingText: "页面加载中",

            /** 表格 */
            metaDataList: [],
            weightNumberList: [],
            weightNumberObject: {},


            /** 编辑对话框 */
            // 数据编辑对话框
            showEditDialog: false,
            currentEditItem: {
                "summation": "1000",
                "sumStandard": "50",
                "gpTime": "2021-12-31",
                "gpData": [
                    {"name": "涨停1", "riseStop": "80"}
                ]
            },
            rules: {
                summation: [{required: true, message: '请输入summation', trigger: 'change'}],
                sumStandard: [{required: true, message: '请输入sumStandard', trigger: 'change'}],
                gpTime: [{required: true, message: '请输入gpTime', trigger: 'change'}],
            },

            // 字段配置对话框
            currentColumnEditItem: {},
            columnRules: {
                name: [{required: true, message: '请输入字段名', trigger: 'change'}],
                minNumber: [{required: true, message: '请输入低极致', trigger: 'change'}],
                maxNumber: [{required: true, message: '请输入高极致', trigger: 'change'}],
                weightNumber: [{required: true, message: '请输入权重', trigger: 'change'}],
            },
            showColumnEditDialog: false,
            columnForm: this.$form.createForm(this),


            /** 图表组件 */
            timer: false,
            screenWidth: document.body.clientWidth,
            chartBox: null,

            // 取消选中的行和列
            unCheckedData: [],
            uncheckedColumn: [],


            /** 其他 */
            darkMode: false,


            currentChartType: "line",
            isStack: false,
            xAxisPosition: "bottom",


        }
    },

    computed: {

        tableDataSource: function () {
            let list = [], itemObject = {};
            for (let i = 0; i < this.metaDataList.length; i++) {
                let item = {};
                try {
                    item = this.metaDataList[i].gpData;
                    itemObject = this.arrayToObject(item)
                } catch (err) {
                    console.error(err);
                }
                list.push({
                    id: this.metaDataList[i].id,
                    gpTime: this.metaDataList[i].gpTime,
                    endSum: this.metaDataList[i].endSum,
                    dataSum: this.metaDataList[i].dataSum,
                    gpDataObject: itemObject,
                    gpData: item,
                    metaData: this.metaDataList[i]
                });
            }
            return list
        },

        tableColumnList: function () {
            let list = [];
            for (let i = 0; i < this.weightNumberList.length; i++) {
                let index = list.findIndex((item) => {
                    return item === this.weightNumberList[i].name
                })
                if (index === -1) {
                    list.push(this.weightNumberList[i].name)
                }
            }
            return list;
        },

        // 因为图表要筛掉没勾选的列，这里使用计算属性过滤
        tableColumnListFiltered: function () {
            let list = [];
            for (let i = 0; i < this.tableColumnList.length; i++) {
                let index = this.uncheckedColumn.findIndex((item) => {
                    return item === this.tableColumnList[i];
                })
                if (index === -1) {
                    list.push(this.tableColumnList[i]);
                }
            }
            return list;
        },

        chartsColumns: function () {
            let columnList = [];
            columnList.push({
                title: "时间",
                dataIndex: "gpTime",
                width: 120,
                align: "center",
                scopedSlots: {
                    customRender: "time",
                }
            })
            for (let i = 0; i < this.tableColumnList.length; i++) {
                columnList.push({
                    title: this.tableColumnList[i],
                    dataIndex: "gpDataObject." + this.tableColumnList[i] + ".riseStop",
                    scopedSlots: {
                        customRender: "riseStop",
                    },
                    align: "center"
                })
            }
            columnList.push({
                title: "总和",
                dataIndex: "endSum",
                align: "center"
            })
            columnList.push({
                title: "操作",
                width: 150,
                align: "center",
                scopedSlots: {
                    customRender: "action",
                }
            })
            return columnList;
        },

        //echarts x值
        allDate: function () {
            let list = [];
            for (let i = 0; i < this.metaDataList.length; i++) {
                if (this.isCheckedDataSource(this.metaDataList[i])) {
                    list.push(this.metaDataList[i].gpTime);
                }
            }
            return list
        },


        // 数据总和
        allDataSeries: function () {
            let seriesList = [];
            for (let i = 0; i < this.tableColumnList.length; i++) {
                // 判断 - 是否在侧边选择展示图形
                if (this.isCheckedColumn(this.tableColumnList[i])) {
                    if (this.currentChartType === 'line') {
                        seriesList.push({
                            type: 'line',
                            name: this.tableColumnList[i],
                            data: this.getDataListByKey(this.tableColumnList[i]),
                            stack: this.isStack ? "0" : null,
                            areaStyle: {
                                opacity: 0.5
                            }
                        })
                    } else {
                        seriesList.push({
                            type: 'bar',
                            name: this.tableColumnList[i],
                            data: this.getDataListByKey(this.tableColumnList[i]),
                            itemStyle: this.isStack ? null : {
                                normal: {
                                    barBorderRadius: [8, 8, 0, 0],
                                },
                                emphasis: {
                                    barBorderRadius: [8, 8, 0, 0],
                                }
                            },
                            stack: this.isStack ? "0" : null
                        })
                    }
                }
            }

            return seriesList;
        },
    },

    methods: {
        /**
         * 获取初始化数据
         */

        // 获取所有行数据
        getServerData: function () {
            let _this = this;
            let queryParam = {
                pageNo: 1,
                pageSize: 50
            }
            this.pageIsLoading = true;
            this.$axios.get(this.baseUrl + "/apcount/list", {params: queryParam}).then((result) => {
                _this.pageIsLoading = false;
                let res = result.data
                if (res.success) {
                    _this.metaDataList = res.result;
                } else {
                    _this.metaDataList = [];
                }
            }).catch((err) => {
                _this.pageIsLoading = false;
                _this.metaDataList = [];
                console.log(err);
            })
        },

        // 获取所有字段
        getAllColumnData: function () {
            let _this = this;
            this.$axios.get(this.baseUrl + "/gpweigh/list", {params: {size: 99999, page: 1}}).then((res) => {
                if (res.data.success) {
                    console.log("获取所有字段。", res.data)
                    _this.weightNumberList = res.data.result.records;
                    let response = this.getWeightObject(res.data.result.records);
                    _this.weightNumberObject = response.object;
                    _this.uncheckedColumn = response.columnUnchecked;
                } else {
                    _this.$message.warn("获取所有字段失败。" + res.data.message);
                }
            }).catch((err) => {
                if (err.response) {
                    _this.$message.error("获取所有字段错误。" + err.response.data.message);
                } else {
                    _this.$message.error("获取所有字段错误。" + err.toString())
                }
            })
        },

        // 根据 key 获取 value
        getTargetData: function (dataList, key, defaultValue) {
            let text = defaultValue
            let index = dataList.findIndex((item) => {
                return item.name === key
            });
            if (index !== -1) {
                text = dataList[index].riseStop;
            }

            return text;
        },

        // 根据 key 获取所有的值
        getDataListByKey: function (key) {
            let list = [];
            if (key === 'endSum') {
                for (let i = 0; i < this.tableDataSource.length; i++) {
                    list.push(this.tableDataSource[i].endSum);
                }
            } else if (key === 'dataSum') {
                for (let i = 0; i < this.tableDataSource.length; i++) {
                    list.push(this.tableDataSource[i].dataSum);
                }
            } else {
                for (let i = 0; i < this.tableDataSource.length; i++) {
                    let dataList = this.tableDataSource[i].gpData;
                    let value = this.getTargetData(dataList, key, 0);
                    list.push(value);
                }
            }

            return list;
        },

        // 根据字段名获取图表类型
        getChartsType: function (key){
            return this.weightNumberObject[key].tbType
        },

        // 数组转对象
        arrayToObject: function (array) {
            let object = {};
            for (let i = 0; i < array.length; i++) {
                object[array[i].name] = array[i]
            }
            return object;
        },

        // 添加一项
        addListItem: function () {
            let currentEditItem = {
                "summation": "1000",
                "sumStandard": "50",
                "gpTime": "2021-12-31",
                "gpData": []
            };
            let tempGPData = [];
            for(let i=0;i< this.tableColumnList.length;i++){
                tempGPData.push({
                    name: this.tableColumnList[i],
                    riseStop: 0,
                })
            }
            currentEditItem.gpData = tempGPData;

            this.currentEditItem = currentEditItem;
            this.showEditDialog = true;
        },

        // 编辑一项
        editItem: function (item) {
            let _item = JSON.parse(JSON.stringify(item));
            console.log("处理前",_item);
            // _item.metaData.gpData = JSON.parse(_item.metaData.gpData);
            let gpData = _item.metaData.gpData, tempGPData = [];
            for(let i=0;i< this.tableColumnList.length;i++){
                console.log(i);
                let index = gpData.findIndex((item) => {
                    console.log(item.name, this.tableColumnList[i])
                    return item.name === this.tableColumnList[i]
                });
                console.log("结果？",index)
                if(index > -1){
                    tempGPData.push({
                        name: gpData[index].name,
                        riseStop: gpData[index].riseStop,
                    })
                }else{
                    tempGPData.push({
                        name: this.tableColumnList[i],
                        riseStop: 0,
                    })
                }
                console.log(index > -1, tempGPData)
            }
            _item.metaData.gpData = tempGPData;
            console.log("处理后",_item);
            this.currentEditItem = _item.metaData;
            this.showEditDialog = true;
        },


        // 为一项添加列
        // addFieldItem: function () {
        //     this.currentEditItem.gpData.push({
        //         "riseStop": "80",
        //         "name": "新建字段" + (this.currentEditItem.gpData.length + 1)
        //     });
        // },

        // 删除字段项目
        // deleteFieldItem: function (index) {
        //     this.currentEditItem.gpData.splice(index, 1);
        // },


        // 保存编辑项目
        saveEditItem: function () {
            let _this = this;
            let queryParam = JSON.parse(JSON.stringify(this.currentEditItem)), queryMethod, queryUrl;
            queryParam.gpData = JSON.stringify(queryParam.gpData);
            if (queryParam.id) {
                queryUrl = "/apcount/edit";
                queryMethod = "PUT";
            } else {
                queryUrl = "/apcount/add";
                queryMethod = "POST";
            }
            //调用接口保存
            this.pageIsLoading = true;
            this.$axios({url: this.baseUrl + queryUrl, data: queryParam, method: queryMethod}).then((res) => {
                _this.pageIsLoading = false;
                console.log(res);
                if (res.data.success) {
                    this.$message.success("保存成功");
                    _this.closeEditDialog();
                    _this.getServerData();
                    _this.getAllColumnData();
                } else {
                    this.$message.warn("保存失败。" + res.data.message);
                }
            }).catch((err) => {
                _this.pageIsLoading = false;
                this.$message.error("保存错误。" + err.toString());
                console.error(err);
            })
        },

        // 关闭编辑对话框
        closeEditDialog: function () {
            this.currentEditItem = {
                "summation": "1000",
                "sumStandard": "50",
                "gpTime": "2021-12-31",
                "gpData": []
            };
            this.showEditDialog = false;
        },


        // 检查此字段是否选中
        isCheckedColumn: function (item) {
            let index = this.uncheckedColumn.findIndex((sItem) => {
                return sItem === item.name
            })
            return (index === -1)
        },

        // 此字段选中变化
        columnCheckedChange: function (e, item) {
            console.log(e, item);
            let index = this.uncheckedColumn.findIndex((sItem) => {
                return sItem === item.name
            })
            if (index > -1) {
                // 将此字段从unChecked 列表移除
                this.uncheckedColumn.splice(index, 1);
            } else {
                // 将字段追加到 unChecked 列表
                this.uncheckedColumn.push(item.name);
            }

        },

        // 字段list转object
        getWeightObject: function (list) {
            let object = {}, columnUnchecked = [];
            for (let i = 0; i < list.length; i++) {
                object[list[i].name] = {
                    weightNumber: list[i].weightNumber,
                    minNumber: list[i].minNumber,
                    maxNumber: list[i].maxNumber,
                    id: list[i].id,
                    isShow: list[i].isShow,
                    name: list[i].name,
                    tbOverlay: list[i].tbOverlay,
                    tbPlace: list[i].tbPlace,
                    tbType: list[i].tbType,
                }
                if(list[i].isShow === '1'){
                    columnUnchecked.push(list[i].name);
                }
            }
            return {object, columnUnchecked};
        },

        // 保存全部列配置
        saveColumnConfig: function () {
            let _this = this;
            // 拼接请求数据（object转数组）
            let array = [];
            for (let i in this.weightNumberObject) {
                array.push({
                    id: this.weightNumberObject[i].id,
                    weightNumber: this.weightNumberObject[i].weightNumber,
                    maxNumber: this.weightNumberObject[i].maxNumber,
                    minNumber: this.weightNumberObject[i].minNumber,
                })
            }
            console.log(array);
            this.pageIsLoading = true;
            this.$axios.put(this.baseUrl + "/gpweigh/editBatch", array).then((res) => {
                _this.pageIsLoading = false;
                if (res.data.success) {
                    _this.$message.success("保存成功");
                    _this.getAllColumnData();
                } else {
                    _this.$message.warn("保存失败。" + res.message);
                }
            }).catch((err) => {
                _this.pageIsLoading = false;
                if (err.response) {
                    _this.$message.error("保存错误。" + err.response.data.message)
                } else {
                    _this.$message.error("保存错误。" + err.toString())
                }
            })
        },

        // 此行数据是否选中
        isCheckedDataSource: function (item) {
            console.log(item);
            let index = this.unCheckedData.findIndex((sItem) => {
                return sItem === item.id
            })
            return index === -1;
        },

        // 此行数据选中变化
        dataSourceCheckedChange: function (e, item) {
            console.log(item);
            item = JSON.parse(JSON.stringify(item));
            item = item.metaData;
            let index = this.unCheckedData.findIndex((sItem) => {
                return sItem === item.id
            })
            if (index > -1) {
                // 将此字段从unChecked 列表移除
                this.unCheckedData.splice(index, 1);
            } else {
                // 将字段追加到 unChecked 列表
                this.unCheckedData.push(item.id);
            }

        },

        // 删除一项
        deleteItem: function (item) {
            let _this = this;
            this.$axios.delete(this.baseUrl + "/apcount/delete", {params: {id: item.metaData.id}}).then((res) => {
                if (res.data.success) {
                    _this.$message.success("删除成功");
                    _this.getServerData();
                    _this.getAllColumnData();
                } else {
                    _this.$message.warn("删除失败。" + res.data.message);
                }
            }).catch((err) => {
                _this.$message.error("删除错误" + err.toString());
            })
        },


        /**
         * 字段管理相关
         */

        // 编辑字段
        editColumnItem: function (record) {
            this.currentColumnEditItem = JSON.parse(JSON.stringify(record));
            this.showColumnEditDialog = true;
        },

        // 保存字段配置
        saveColumnEditItem: function () {
            let _this = this;
            this.$refs.columnForm.validate((success, object) => {
                console.log(success, object);
                if(success){
                    let queryUrl, queryParam, queryMethod;
                    queryParam = JSON.parse(JSON.stringify(this.currentColumnEditItem));
                    if(queryParam.id){
                        queryParam.isShow = this.isCheckedColumn(queryParam)?0:1;
                    }
                    // 检查添加还是编辑
                    if (queryParam.id) {
                        // 编辑
                        queryUrl = this.baseUrl + "/gpweigh/edit";
                        queryMethod = "PUT";
                    } else {
                        // 添加
                        queryUrl = this.baseUrl + "/gpweigh/add";
                        queryMethod = "POST";
                    }
                    this.$axios({url: queryUrl, data: queryParam, method: queryMethod}).then((res) => {
                        if (res.data.success) {
                            console.log("保存字段配置。", res.data)
                            _this.$message.success("保存成功");
                            _this.closeColumnEditDialog();
                            _this.getAllColumnData();
                            _this.getServerData();
                        } else {
                            _this.$message.warn("保存字段配置失败。" + res.data.message);
                        }
                    }).catch((err) => {
                        if (err.response) {
                            _this.$message.error("保存字段配置错误。" + err.response.data.message);
                        } else {
                            _this.$message.error("保存字段配置错误。" + err.toString())
                        }
                    })
                }
            })

        },

        // 删除字段
        deleteColumnItem: function (item) {
            let _this = this;
            this.$axios.delete(this.baseUrl + "/gpweigh/delete", {params: {id: item.id}}).then((res) => {
                if (res.data.success) {
                    _this.$message.success("删除成功");
                    _this.getAllColumnData();
                    _this.getServerData();
                } else {
                    _this.$message.warn("删除失败。" + res.data.message);
                }
            }).catch((err) => {
                if (err.response) {
                    _this.$message.error("删除错误。" + err.response.data.message);
                } else {
                    _this.$message.error("删除失败。" + err.toString());
                }
            })
        },

        // 关闭字段配置对话框
        closeColumnEditDialog: function () {
            this.currentColumnEditItem = {};
            // this.columnForm.resetFields();
            this.showColumnEditDialog = false;
        },

    },

    created: function () {
        this.getAllColumnData();
        this.getServerData()
    },
}
</script>

<style scoped lang="less">
html, body, #app {
    padding: 0;
    margin: 0;
}

.page_loader_box {
    width: 100%;
    height: 100vh;
    position: fixed;
    z-index: 99;
    background-color: rgba(0, 0, 0, 0.5);
    display: flex;
    align-items: center;
    justify-content: center;

    .page_loader {
        width: 250px;
        height: 200px;
        background: rgba(255, 255, 255, 0.5);
        position: relative;
        display: flex;
        flex-flow: column;
        align-items: center;
        justify-content: center;
        border-radius: 20px;

        .text {
            margin-top: 50px;
            color: #ffffff;
        }
    }
}


.line-spin-fade-loader {
    position: relative;
    margin-top: 20px;
    margin-left: -10px;

    > div {
        border-radius: 2px;
        margin: 2px;
        background-color: #fff;
        -webkit-animation-fill-mode: both;
        animation-fill-mode: both;
        position: absolute;
        width: 5px;
        height: 15px;

        &:nth-child(1) {
            top: 20px;
            left: 0;
            animation: line-spin-fade-loader 1.2s -.84s infinite ease-in-out
        }

        &:nth-child(2) {
            top: 13.64px;
            left: 13.64px;
            -webkit-transform: rotate(-45deg);
            transform: rotate(-45deg);
            animation: line-spin-fade-loader 1.2s -.72s infinite ease-in-out
        }

        &:nth-child(3) {
            top: 0;
            left: 20px;
            -webkit-transform: rotate(90deg);
            transform: rotate(90deg);
            animation: line-spin-fade-loader 1.2s -.6s infinite ease-in-out
        }

        &:nth-child(4) {
            top: -13.64px;
            left: 13.64px;
            -webkit-transform: rotate(45deg);
            transform: rotate(45deg);
            animation: line-spin-fade-loader 1.2s -.48s infinite ease-in-out
        }

        &:nth-child(5) {
            top: -20px;
            left: 0;
            animation: line-spin-fade-loader 1.2s -.36s infinite ease-in-out
        }

        &:nth-child(6) {
            top: -13.64px;
            left: -13.64px;
            -webkit-transform: rotate(-45deg);
            transform: rotate(-45deg);
            animation: line-spin-fade-loader 1.2s -.24s infinite ease-in-out
        }

        &:nth-child(7) {
            top: 0;
            left: -20px;
            -webkit-transform: rotate(90deg);
            transform: rotate(90deg);
            animation: line-spin-fade-loader 1.2s -.12s infinite ease-in-out
        }

        &:nth-child(8) {
            top: 13.64px;
            left: -13.64px;
            -webkit-transform: rotate(45deg);
            transform: rotate(45deg);
            animation: line-spin-fade-loader 1.2s 0s infinite ease-in-out
        }
    }
}


#app {
    width: 100vw;
    height: 100vh;
    background-color: #ffffff;
    color: #000000;
    display: flex;
    //padding: 10px;
    box-sizing: border-box;
    transition: background-color 200ms ease-in-out, color 200ms ease-in-out;

    .left_input_box {
        width: 400px;
        border-right: 2px solid #d9d9d9;
        display: flex;
        flex-flow: column;
        transition: border 200ms ease-in-out;
        padding: 10px;

        table {
            border: 1px solid #d9d9d9;

            th, td {
                border-right: 1px solid #d9d9d9;
                border-bottom: 1px solid #d9d9d9;
            }
        }
    }

    .right_display_box {
        width: 300px;
        flex: auto;
        //border: 2px solid #000000;
        display: flex;
        flex-flow: column;
        transition: border 200ms ease-in-out;

        .table_box {
            height: 300px;
            overflow: auto;
            border-bottom: 2px solid #d9d9d9;
            transition: border 200ms ease-in-out;
            padding: 10px;
        }

        .chart_box {
            width: 100%;
            height: calc(100vh - 20px - 300px);
            overflow-y: auto;
            overflow-x: hidden;
            display: flex;
            flex-wrap: wrap;
            align-content: flex-start;
            flex: auto;

            .charts_item {
                width: 100%;
                height: 500px;

            }
        }
    }


}

.form_item_box {
    display: flex;
    margin: 5px 0;

    .label {
        width: 156px;
        font-size: 14px;
        color: rgba(0, 0, 0, 0.85);
        display: flex;
        align-items: center;
        justify-content: flex-end;
    }

    .input {
        width: 60%;
        flex: auto;
    }

    .gp_data_list {
        width: 60%;
        flex: auto;

        .gp_data_item {
            border: 1px solid #d9d9d9;
            margin: 10px 0 10px 0;
            padding: 20px;
        }
    }
}

.form_box {
    //margin: 0px 0;

    .item {
        display: flex;
        align-items: center;
        margin: 10px 0;

        .label {
            width: 110px;
            text-align: right;

            &:after {
                content: ":";
                margin: 0 10px 0 5px;
            }
        }

        .input {
            width: calc(100% - 110px);
            flex: auto;
        }
    }
}


@keyframes line-spin-fade-loader {
    50% {
        opacity: .3
    }
    100% {
        opacity: 1
    }
}

</style>
