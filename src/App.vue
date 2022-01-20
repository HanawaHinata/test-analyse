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


                <div style="font-size: 12pt;margin: 40px 0 0 0;font-weight: bold;">总和设置</div>
                <a-form-model ref="ruleForm" :model="countDataFormData" :rules="rules"
                              :label-col="{ span: 7 }" :wrapper-col="{ span: 12 }">
                    <div class="form_box">

                        <a-form-model-item label="指数起步值" style="margin: 0">
                            <a-input-number style="width: 100%;" v-model="countDataFormData.countData"/>
                        </a-form-model-item>


                        <a-form-model-item label="冷暖研判指标" style="margin: 0">
                            <a-input-number style="width: 100%;" v-model="countDataFormData.standard"/>
                        </a-form-model-item>

                        <a-button type="primary" icon="save" @click="saveCountConfig"
                                  style="margin-top: 10px;width: 100%;">保存指标
                        </a-button>
                    </div>
                </a-form-model>

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
                    <a-button class="addbutton" style="display: block; margin: 10px auto;" @click="addListItem"
                              type="primary" icon="plus">添加
                    </a-button>

                    <a-table size="middle" :columns="chartsColumns" rowKey="id" :dataSource="tableDataAsc" bordered
                             :pagination="false" :sortDirections="['descend', 'ascend']" :scroll="{ x: 3500, y: 280 }">
                        <template slot="riseStop" slot-scope="text">
                            <a-tooltip>
                                <template slot="title">
                                    {{ text.data == null || text.data === '' ? 0 : text.data }}
                                </template>
                                {{ text.riseStop == null || text.riseStop === '' ? 0 : text.riseStop }}
                            </a-tooltip>
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

                </div>
                <div class="chart_box" id="chartBox">
                    <item-charts class="charts_item_qx" title="情绪指数" index="endSum"
                                 :x-data="allDate" :y-data="getDataListByKey('endSum')"/>
                    <item-charts class="charts_item" title="情绪温度" index="dataSum"
                                 :x-data="allDate" :y-data="getDataListByKey('dataSum')"/>
                    <template v-for="(item, index) in tableColumnListFiltered">
                        <!--                        {{getChartsType(item)}}-->
                        <item-charts :key="index" :index="index" class="charts_item" :title="item"
                                     :type="getChartsType(item)"
                                     :x-data="allDate" :y-data="getDataListByKey(item)"/>
                    </template>
                    <mixin-charts class="charts_item" style="height: 500px;" title="总合数据图" index="dataAll"
                                  :type="currentChartType"
                                  :xAxisPosition="xAxisPosition" :x-data="allDate" :series="allDataSeries"/>

                </div>
            </div>

            <a-modal :title="(currentEditItem.id?'编辑':'新建')+'行数据'" :visible="showEditDialog" :width="800"
                     @ok="saveEditItem" @cancel="closeEditDialog" :confirm-loading="saveLoading">
                <a-spin :spinning="saveLoading">
                    <a-form-model ref="ruleForm" :model="currentEditItem" :rules="rules"
                                  :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }">
                        <a-form-model-item label="时间" prop="gpTime" style="margin: 0">
                            <a-input v-model="currentEditItem.gpTime"/>
                        </a-form-model-item>

                        <template v-for="(item, index) in currentEditItem.gpData">
                            <a-form-model-item :key="index" :label="item.name" style="margin: 0">
                                <a-input v-model="item.riseStop" placeholder="0"/>
                            </a-form-model-item>
                        </template>

                    </a-form-model>
                </a-spin>
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
            // baseUrl: "//127.0.0.1:8072/wealth",
            pageIsLoading: true,
            pageLoadingText: "页面加载中",

            /** 表格 */
            metaDataList: [],
            weightNumberList: [],
            weightNumberObject: {},
            weightNumbersss: [1, 2, 1, 2, 12, 1, 2, 11, 2, 3, 2, 32, 32, 32, 32, 323, 2],


            //获取总合数据
            countDataFormData: {
                countData: "",
                standard: ""
            },

            //获取当前时间
            upgpTime: "",

            /** 编辑对话框 */
            // 数据编辑对话框
            saveLoading: false,
            showEditDialog: false,
            currentEditItem: {
                "summation": "1000",
                "sumStandard": "50",
                "gpTime": "",
                "gpData": [
                    {"name": "涨停1", "riseStop": "80", "id": ""}
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
            currentCherTypeList: [],
            isStack: false,
            xAxisPosition: "bottom",


        }
    },

    computed: {

        // 表格数据
        tableDataSource: function () {
            let list = [], itemObject = {};
            for (let i = 0; i < this.metaDataList.length; i++) {
                // for (let i = this.metaDataList.length-1; i>=0; i--) {
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

        // 这是啥？也是表格数据？
        tableDataAsc: function () {
            let list = [], itemObject = {};

            // for (let i = 0; i < this.metaDataList.length; i++) {
            for (let i = this.metaDataList.length - 1; i >= 0; i--) {
                let dataOnes = [];
                let item = {};
                try {
                    item = this.metaDataList[i].gpData;
                    // for(let a=0;a<=this.metaDataList[i].gpData.length;a++){
                    //
                    //   let datase= this.metaDataList[i].gpData[a].data;
                    //   dataOnes.push(datase)
                    // }
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
                    datas: dataOnes,
                    metaData: this.metaDataList[i]
                });
            }
            return list
        },

        // 列元数据转数组
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

        // 这又是啥？为什么单词是错的？
        tableCheatrsList: function () {
            let list = [];
            for (let i = 0; i < this.weightNumberList.length; i++) {
                list.push({
                    tbType: this.weightNumberList[i].tbType,
                    name: this.weightNumberList[i].name
                })

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

        // 图表字段？这内容看起来像表格字段
        chartsColumns: function () {
            let columnList = [];
            columnList.push({
                title: "时间",
                dataIndex: "gpTime",
                width: 120,
                align: "center",
                fixed: 'left',
                scopedSlots: {
                    customRender: "time",
                }
            })
            // 遍历字段元数据，拼接 gpData 内的字段
            for (let i = 0; i < this.weightNumberList.length; i++) {
                columnList.push({
                    title: this.weightNumberList[i].name,
                    dataIndex: "gpDataObject." + this.weightNumberList[i].id,
                    // dataIndex: "gpDataObject." + this.tableColumnList[i] + ".data",
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
                title: "计算后总和",
                dataIndex: "dataSum",
                align: "center"
            })
            columnList.push({
                title: "操作",
                width: 150,
                align: "center",
                fixed: 'right',
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
            for (let i = 0; i < this.tableCheatrsList.length; i++) {
                // 判断 - 是否在侧边选择展示图形
                if (this.isCheckedColumn(this.tableCheatrsList[i].name)) {
                    // if (this.currentChartType === 'line') {
                    if (this.tableCheatrsList[i].tbType === 'line') {
                        seriesList.push({
                            type: 'line',
                            name: this.tableCheatrsList[i].name,
                            data: this.getDataListByKey(this.tableCheatrsList[i].name),
                            stack: this.isStack ? "0" : null
                            // areaStyle: this.isStack ? {
                            //     opacity: 0
                            // } : null
                        })
                    } else {
                        seriesList.push({
                            type: 'bar',
                            name: this.tableCheatrsList[i].name,
                            data: this.getDataListByKey(this.tableCheatrsList[i].name),
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
         * 获取当前时间
         * */
        addDate() {
            const nowDate = new Date();
            const date = {
                year: nowDate.getFullYear(),
                month: nowDate.getMonth() + 1,
                date: nowDate.getDate(),
            }
            const newmonth = date.month > 10 ? date.month : '0' + date.month
            const day = date.date > 10 ? date.date : '0' + date.date
            this.upgpTime = date.year + '-' + newmonth + '-' + day

        },


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

        // 获取总合数据
        getCountData: function () {
            let _this = this;
            let queryParam = {
                pageNo: 1,
                pageSize: 50
            }
            this.pageIsLoading = true;
            this.$axios.get(this.baseUrl + "/apcount/listCount", {params: queryParam}).then((result) => {
                _this.pageIsLoading = false;
                let res = result.data
                if (res.success) {
                    _this.countDataFormData = res.result.records[0];
                } else {
                    _this.countDataFormData = {
                        countData: "",
                        standard: ""
                    };

                }
            }).catch((err) => {
                _this.pageIsLoading = false;
                _this.countDataFormData = {
                    countData: "",
                    standard: ""
                };
                console.log(err);
            })
        },

        // 获取所有字段
        getAllColumnData: function () {
            let _this = this;
            this.$axios.get(this.baseUrl + "/gpweigh/list", {params: {size: 99999, page: 1}}).then((res) => {
                if (res.data.success) {
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

        // 根据 所有图表type属性 获取所有的值
        getChartsByKey: function (key) {
            let list = [];
            if (key === 'tbType') {
                for (let i = 0; i < this.tableColumnList.length; i++) {
                    list.push(this.tableColumnList[i].tbType);
                }
            } else if (key === 'tbOverlay') {
                for (let i = 0; i < this.tableColumnList.length; i++) {
                    list.push(this.tableColumnList[i].tbOverlay);
                }
            }

            return list;
        },

        // 根据字段名获取图表类型
        getChartsType: function (key) {
            return this.weightNumberObject[key].tbType
        },

        // 数组转对象
        arrayToObject: function (array) {
            let object = {};
            for (let i = 0; i < array.length; i++) {
                object[array[i].id] = array[i]
            }
            return object;
        },

        // 添加一项
        addListItem: function () {
            let currentEditItem = {
                "summation": "1000",
                "sumStandard": "50",
                "gpTime": this.upgpTime,
                "gpData": []
            };
            // 初始化 gpData 。初始化时，直接复用字段表数据生成一个全是 0 的数组
            let tempGPData = [];
            for (let i = 0; i < this.weightNumberList.length; i++) {
                tempGPData.push({
                    name: this.weightNumberList[i].name,
                    id: this.weightNumberList[i].id,
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
            console.log("处理前", _item);
            // 处理 gpData。有两种情况，一种是gpData为空，另一种是之前存在gpData。
            // 为空的话跟添加一样直接初始化，不为空的话就重写为干净的数组。
            let gpData = _item.metaData.gpData,
                tempGPData = [];
            // 遍历字段表元数据
            for (let i = 0; i < this.weightNumberList.length; i++) {
                // 根据 ID 找出gpData中是否存在此字段对应的值，存在就取出来，不存在就默认 0
                let index = gpData.findIndex((item) => {
                    return item.id === this.weightNumberList[i].id
                });
                tempGPData.push({
                    name: this.weightNumberList[i].name,
                    id: this.weightNumberList[i].id,
                    riseStop: index > -1?gpData[index].riseStop:0,
                })
            }
            _item.metaData.gpData = tempGPData;
            console.log("处理后", _item);
            this.currentEditItem = _item.metaData;
            this.showEditDialog = true;
        },

        // 保存编辑项目
        saveEditItem: function () {
            let _this = this;
            let queryParam = JSON.parse(JSON.stringify(this.currentEditItem)), queryMethod, queryUrl;
            queryParam.gpData = JSON.stringify(queryParam.gpData);
            console.log(queryParam);
            if (queryParam.id) {
                queryUrl = "/apcount/edit";
                queryMethod = "PUT";
            } else {
                queryUrl = "/apcount/add";
                queryMethod = "POST";
            }
            // 调用接口保存
            this.saveLoading = true;
            this.$axios({url: this.baseUrl + queryUrl, data: queryParam, method: queryMethod}).then((res) => {
                _this.saveLoading = false;
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
                if (list[i].isShow === '1') {
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
                    location.reload()
                    _this.$router.go(0)
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
        // 保存总合数据
        saveCountConfig: function () {
            let _this = this;
            console.log(_this.countDataFormData);
            this.pageIsLoading = true;
            this.$axios.put(this.baseUrl + "/apcount/editCount", _this.countDataFormData).then((res) => {
                _this.pageIsLoading = false;
                if (res.data.success) {
                    _this.$message.success("保存成功");
                    _this.getCountData();
                    location.reload();
                    _this.$router.go(0);
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
            // console.log(item);
            let index = this.unCheckedData.findIndex((sItem) => {
                return sItem === item.id
            })
            return index === -1;
        },

        // 此行数据选中变化
        dataSourceCheckedChange: function (e, item) {
            // console.log(item);
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
                if (success) {
                    let queryUrl, queryParam, queryMethod;
                    queryParam = JSON.parse(JSON.stringify(this.currentColumnEditItem));
                    if (queryParam.id) {
                        queryParam.isShow = this.isCheckedColumn(queryParam) ? 0 : 1;
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
        this.getServerData();
        this.getCountData();
        this.addDate();
    },
}
</script>

<style scoped lang="less">
html, body, #app {
    padding: 0;
    margin: 0;
}

.addbutton {
    height: 30px;
}

.jstype {
    background-color: #69c0ff;
}


#app {
    width: 100vw;
    //height: 100vh;
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
        height: 100vh;
        overflow: auto;

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
        height: 100vh;

        .table_box {
            height: 420px;
            overflow-y: auto;
            overflow-x: hidden;
            border-bottom: 2px solid #d9d9d9;
            transition: border 200ms ease-in-out;
            padding: 10px;
        }

        .chart_box {
            width: 100%;
            height: calc(100vh - 20px - 420px);
            overflow-y: auto;
            overflow-x: hidden;
            display: flex;
            flex-wrap: wrap;
            align-content: flex-start;
            flex: auto;

            .charts_item {
                width: 100%;
                height: 200px;
            }

            .charts_item_qx {
                width: 100%;
                height: 300px;
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


</style>
