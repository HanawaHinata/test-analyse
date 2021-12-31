<template>
    <a-spin :spinning="pageIsLoading">
        <div id="app">
            <div class="right_display_box">
                <div class="table_box">
                    <a-table size="middle" :columns="chartsColumns" rowKey="id" :dataSource="tableDataSource" bordered
                             :pagination="false">
                        <div slot="filterDropdown"
                             slot-scope="{setSelectedKeys, selectedKeys, confirm, clearFilters, column }"
                             style="padding: 8px">
                            <div>
                                是否在ECharts中显示此列数据：
                                <a-checkbox :checked="isCheckedColumn(column.title)"
                                            @change="columnCheckedChange($event, column.title)"
                                            title="是否在ECharts中显示此列数据"/>
                            </div>
                            <div style="margin-top: 5px;">
                                权重值：
                                <a-input-number v-model="weightNumberObject[column.title]" placeholder="0"/>
                                <a-button type="primary" icon="save" size="small"
                                          @click="saveWeightNumber(column.title)"
                                          style="margin-left: 5px;">保存
                                </a-button>
                            </div>
                        </div>
                        <a-icon
                            slot="filterIcon"
                            slot-scope="filtered"
                            type="down-circle"
                            :style="{ color: filtered ? '#108ee9' : undefined }"
                        />
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
                    <template v-for="(item, index) in tableColumnListFiltered">
                        <line-charts :key="index" :index="index" class="charts_item" :title="item"
                                     :x-data="allDate" :y-data="getDataListByKey(item)"/>
                    </template>

                    <line-charts class="charts_item" title="总和" index="endSum"
                                 :x-data="allDate" :y-data="getDataListByKey('endSum')"/>
                </div>
            </div>

            <a-modal :title="(currentEditItem.id?'编辑':'新建')+'行数据'" :visible="showEditDialog" :width="800"
                     @ok="saveEditItem" @cancel="closeEditDialog">
                <a-form-model ref="ruleForm" :model="currentEditItem" :rules="rules"
                              :label-col="{ span: 5 }" :wrapper-col="{ span: 19 }">
                    <a-form-model-item label="summation" prop="summation" style="margin: 0">
                        <a-input-number style="width: 100%;" v-model="currentEditItem.summation"/>
                    </a-form-model-item>
                    <a-form-model-item label="sumStandard" prop="sumStandard" style="margin: 0">
                        <a-input-number style="width: 100%;" v-model="currentEditItem.sumStandard"/>
                    </a-form-model-item>
                    <a-form-model-item label="gpTime" prop="gpTime" style="margin: 0">
                        <a-input v-model="currentEditItem.gpTime"/>
                    </a-form-model-item>

                    <div class="form_item_box">
                        <div class="label">下属字段：</div>
                        <div class="gp_data_list">
                            <template v-for="(item, index) in currentEditItem.gpData">
                                <div class="gp_data_item" :key="index">
                                    <a-form-model-item label="字段名" prop="gpDataName" style="margin: 0">
                                        <a-input v-model="item.name"/>
                                    </a-form-model-item>
                                    <!--                                    <a-form-model-item label="weightNumber" prop="gpDataWeightNumber" style="margin: 0">-->
                                    <!--                                        <a-input-number style="width: 100%;"  v-model="item.weightNumber"/>-->
                                    <!--                                    </a-form-model-item>-->
                                    <a-form-model-item label="minNumber" prop="gpDataMinNumber" style="margin: 0">
                                        <a-input-number style="width: 100%;" v-model="item.minNumber"/>
                                    </a-form-model-item>
                                    <a-form-model-item label="maxNumber" prop="gpDataMaxNumber" style="margin: 0">
                                        <a-input-number style="width: 100%;" v-model="item.maxNumber"/>
                                    </a-form-model-item>
                                    <a-form-model-item label="riseStop" prop="gpDataRiseStop" style="margin: 0">
                                        <a-input-number style="width: 100%;" v-model="item.riseStop"/>
                                    </a-form-model-item>
                                    <a-button type="danger" icon="delete" @click="deleteFieldItem(index)">删除字段项
                                    </a-button>
                                </div>
                            </template>
                            <a-button type="primary" icon="plus" @click="addFieldItem">添加字段项</a-button>
                        </div>
                    </div>

                </a-form-model>
            </a-modal>

        </div>
    </a-spin>
</template>

<script>
// let mapChart = null;
// 引入基本模板
import LineCharts from "./components/lineCharts";


export default {
    name: 'App',
    components: {LineCharts},
    data() {
        return {
            baseUrl: "//139.224.41.178:8073/wealth",
            pageIsLoading: true,
            pageLoadingText: "页面加载中",

            metaDataList: [],


            //编辑对话框
            showEditDialog: false,
            currentEditItem: {
                "summation": "1000",
                "sumStandard": "50",
                "gpTime": "2021-12-31",
                "gpData": [
                    {"name": "涨停1", "weightNumber": "0", "riseStop": "80", "maxNumber": "24", "minNumber": "100"}
                ]
            },
            weightNumberList: [],
            weightNumberObject: {},
            rules: {
                summation: [{required: true, message: '请输入summation', trigger: 'change'}],
                sumStandard: [{required: true, message: '请输入sumStandard', trigger: 'change'}],
                gpTime: [{required: true, message: '请输入gpTime', trigger: 'change'}],
                // gpDataName: [{required: true, message: '请输入字段名', trigger: 'change'}],
                // gpDataWeightNumber: [{required: true, message: '请输入WeightNumber', trigger: 'change'}],
                // gpDataMinNumber: [{required: true, message: '请输入minNumber', trigger: 'change'}],
                // gpDataMaxNumber: [{required: true, message: '请输入maxNumber', trigger: 'change'}],
                // gpDataRiseStop: [{required: true, message: '请输入riseNumber', trigger: 'change'}],
            },

            timer: false,
            screenWidth: document.body.clientWidth,
            chartBox: null,

            darkMode: false,

            unCheckedData: [],
            uncheckedColumn: [],

        }
    },

    computed: {

        tableDataSource: function () {
            let list = [], itemObject = {};
            for (let i = 0; i < this.metaDataList.length; i++) {
                let item = {};
                try {
                    item = JSON.parse(this.metaDataList[i].gpData);
                    itemObject = this.arrayToObject(item)
                } catch (err) {
                    console.error(err);
                }
                list.push({
                    id: this.metaDataList[i].id,
                    gpTime: this.metaDataList[i].gpTime,
                    endSum: this.metaDataList[i].endSum,
                    gpDataObject: itemObject,
                    gpData: item,
                    metaData: this.metaDataList[i]
                });
            }
            return list
        },

        tableColumnList: function () {
            let list = [];
            for (let i = 0; i < this.tableDataSource.length; i++) {
                for (let j = 0; j < this.tableDataSource[i].gpData.length; j++) {
                    let index = list.findIndex((item) => {
                        return item === this.tableDataSource[i].gpData[j].name
                    })
                    if (index === -1) {
                        list.push(this.tableDataSource[i].gpData[j].name)
                    }
                }
            }
            return list;
        },

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
                    dataIndex: "gpDataObject." + this.tableColumnList[i] + ".data",
                    scopedSlots: {
                        filterIcon: 'filterIcon',
                        filterDropdown: 'filterDropdown',
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

        allDate: function () {
            let list = [];
            for (let i = 0; i < this.metaDataList.length; i++) {
                if (this.isCheckedDataSource(this.metaDataList[i])) {
                    list.push(this.metaDataList[i].gpTime);
                }
            }
            return list
        },

    },

    methods: {
        // 获取所有数据
        getServerData: function () {
            let _this = this;
            let queryParam = {
                page: 1,
                size: 99999
            }
            this.pageIsLoading = true;
            this.$axios.get(this.baseUrl + "/apcount/list", {params: queryParam}).then((result) => {
                _this.pageIsLoading = false;
                let res = result.data
                if (res.success) {
                    _this.metaDataList = res.result.records;
                } else {
                    _this.metaDataList = [];
                }
            }).catch((err) => {
                _this.pageIsLoading = false;
                _this.metaDataList = [];
                console.log(err);
            })
        },

        // 根据 key 获取 value
        getTargetData: function (dataList, key, defaultValue) {
            let text = defaultValue
            let index = dataList.findIndex((item) => {
                return item.name === key
            });
            if (index !== -1) {
                text = dataList[index].data;
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
            } else {
                for (let i = 0; i < this.tableDataSource.length; i++) {
                    let dataList = this.tableDataSource[i].gpData;
                    let value = this.getTargetData(dataList, key, 0);
                    list.push(value);
                }
            }

            return list;
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
            this.currentEditItem = {
                "summation": "1000",
                "sumStandard": "50",
                "gpTime": "2021-12-31",
                "gpData": [
                    {"name": "新建字段1", "weightNumber": "0", "riseStop": "80", "maxNumber": "24", "minNumber": "100"}
                ]
            };
            this.showEditDialog = true;
        },

        // 编辑一项
        editItem: function (item) {
            let _item = JSON.parse(JSON.stringify(item));
            console.log(item);
            _item.metaData.gpData = JSON.parse(_item.metaData.gpData);
            this.currentEditItem = _item.metaData;
            this.showEditDialog = true;
        },

        // 为一项添加列
        addFieldItem: function () {
            this.currentEditItem.gpData.push({
                "weightNumber": "80",
                "minNumber": "80",
                "riseStop": "80",
                "maxNumber": "80",
                "name": "新建字段" + (this.currentEditItem.gpData.length + 1)
            });
        },

        // 删除字段项目
        deleteFieldItem: function (index) {
            this.currentEditItem.gpData.splice(index, 1);
        },


        // 保存编辑项目
        saveEditItem: function () {
            let _this = this;

            //调用接口保存
            this.pageIsLoading = true;
            this.$axios.post(this.baseUrl + "/apcount/add", this.currentEditItem).then((res) => {
                _this.pageIsLoading = false;
                console.log(res);
                if (res.data.success) {
                    this.$message.success("保存成功");
                    _this.closeEditDialog();
                    _this.getServerData();
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
                "gpData": [
                    {"name": "涨停1", "weightNumber": "0", "riseStop": "80", "maxNumber": "24", "minNumber": "100"}
                ]
            };
            this.showEditDialog = false;
        },


        // 切换日间、夜间模式
        changeThemeColor: function () {
            const domTarget = document.querySelector('#app');

            if (this.darkMode) {
                domTarget.style.setProperty('--background_color', '#ffffff');
                domTarget.style.setProperty('--text_color', '#000000');
            } else {
                domTarget.style.setProperty('--background_color', '#000000');
                domTarget.style.setProperty('--text_color', '#59f6fb');
            }

            this.darkMode = !this.darkMode;

        },

        // 检查此字段是否选中
        isCheckedColumn: function (item) {
            let index = this.uncheckedColumn.findIndex((sItem) => {
                return sItem === item
            })
            return index === -1
        },

        // 此字段选中变化
        columnCheckedChange: function (e, item) {
            console.log(e, item);
            let index = this.uncheckedColumn.findIndex((sItem) => {
                return sItem === item
            })
            if (index > -1) {
                // 将此字段从unChecked 列表移除
                this.uncheckedColumn.splice(index, 1);
            } else {
                // 将字段追加到 unChecked 列表
                this.uncheckedColumn.push(item);
            }

        },

        // 获取所有权重值
        getAllWeightNumber: function () {
            let _this = this;
            this.$axios.get(this.baseUrl + "/gpweigh/list", {size: 99999}).then((res) => {
                if (res.data.success) {
                    console.log("获取权值。", res.data)
                } else {
                    _this.$message.warn("获取权值失败。" + res.data.message);
                }
            }).catch((err) => {
                if (err.response) {
                    _this.$message.error("获取权值错误。" + err.response.data.message);
                } else {
                    _this.$message.error("获取权值错误。" + err.toString())
                }
            })
        },

        // 权重值保存
        saveWeightNumber: function (columnName) {
            let _this = this;
            let weightNumber = this.weightNumberObject[columnName];
            if (weightNumber == null || weightNumber === "") {
                weightNumber = 0
            }
            // 检查添加还是编辑
            let index = this.weightNumberList.findIndex((item) => {
                return item.column === columnName
            });
            let queryUrl, queryParam, queryMethod;
            queryParam = {
                name: columnName,
                weightNumber: weightNumber
            }
            if (index > -1) {
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
                    console.log("保存权值。", res.data)
                    _this.getAllWeightNumber();
                    _this.getServerData()
                } else {
                    _this.$message.warn("保存权值失败。" + res.data.message);
                }
            }).catch((err) => {
                if (err.response) {
                    _this.$message.error("保存权值错误。" + err.response.data.message);
                } else {
                    _this.$message.error("保存权值错误。" + err.toString())
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
            this.$axios.delete(this.baseUrl + "/delete", {params: {id: item.metaData.id}}).then((res) => {
                if (res.data.success) {
                    _this.$message.success("删除成功");
                    _this.getServerData()
                } else {
                    _this.$message.warn("删除失败。" + res.data.message);
                }
            }).catch((err) => {
                _this.$message.error("删除错误" + err.toString());
            })
        },
    },

    created: function () {
        this.getAllWeightNumber();
        this.getServerData()
    },
}
</script>

<style scoped lang="less">
* {
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
        width: 300px;
        border-left: 2px solid #000000;
        border-top: 2px solid #000000;
        border-bottom: 2px solid #000000;
        display: flex;
        flex-flow: column;
        transition: border 200ms ease-in-out;

        textarea {
            width: 100%;
            height: 500px;
            flex: auto;
            border: none;
            outline: none;
            padding: 10px;
            box-sizing: border-box;
            background: none;
            color: #000000;
            resize: none;
            transition: color 200ms ease-in-out;
        }

        .control_box {
            display: flex;
            flex-wrap: wrap;
            border-top: 2px solid #000000;
            transition: border 200ms ease-in-out;

            .button {
                flex: auto;
                height: 50px;
                display: flex;
                align-items: center;
                justify-content: center;
                cursor: pointer;
                user-select: none;

                &.add {
                    width: 100%;
                    border-bottom: 2px solid #000000;
                    transition: border 200ms ease-in-out;
                }

                &.cancel {
                    border-right: 2px solid #000000;
                    transition: border 200ms ease-in-out;
                }

                &.dark_mode {
                    width: 100%;
                    border-top: 2px solid #000000;
                    transition: border 200ms ease-in-out;
                }

                &:hover {
                    color: #ffffff;
                    background: #000000;
                    transition: background-color 200ms ease-in-out, color 200ms ease-in-out;
                }
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
            overflow: auto;
            display: flex;
            flex-wrap: wrap;
            align-content: flex-start;

            .charts_item {
                width: 25%;
                height: 250px;
            }
        }
    }


}

.form_item_box {
    display: flex;

    .label {
        width: 156px;
        height: 60px;
        font-size: 14px;
        color: rgba(0, 0, 0, 0.85);
        display: flex;
        align-items: center;
        justify-content: flex-end;
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


@keyframes line-spin-fade-loader {
    50% {
        opacity: .3
    }
    100% {
        opacity: 1
    }
}

</style>
