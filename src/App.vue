<template>
    <div id="app">
<!--        <div class="left_input_box">-->
<!--            <textarea v-model="currentEditItemMetaData"></textarea>-->
<!--            <div class="control_box">-->
<!--                <div class="button add" @click="addFieldItem">添加数据列</div>-->
<!--                <div class="button cancel" @click="cancelEditItem">重置编辑</div>-->
<!--                <div class="button save" @click="saveEditItem">保存编辑</div>-->
<!--                <div class="button dark_mode" @click="changeThemeColor">切换{{ darkMode ? '日间' : '夜间' }}模式</div>-->
<!--            </div>-->
<!--        </div>-->
        <div class="right_display_box">
            <div class="table_box">
<!--                <table>-->
<!--                    <tr>-->
<!--                        <th>时间</th>-->
<!--                        <template v-for="(item, index) in tableColumnList">-->
<!--                            <th :key="index">-->
<!--                                <input title="是否在ECharts中显示此列数据" type="checkbox"-->
<!--                                       :checked="isCheckedColumn(item)"-->
<!--                                       @change="columnCheckedChange($event, item)"/>-->
<!--                                {{ item }}-->
<!--                            </th>-->
<!--                        </template>-->
<!--                        <th>操作</th>-->
<!--                    </tr>-->
<!--                    <template v-for="(item, index) in tableDataSource">-->
<!--                        <tr :key="index">-->
<!--                            <td>-->
<!--                                <input title="是否在ECharts中显示行数据" type="checkbox"-->
<!--                                       :checked="isCheckedDataSource(item)"-->
<!--                                       @change="dataSourceCheckedChange($event, item)"/>-->
<!--                                {{ item.gpTime }}-->
<!--                            </td>-->
<!--                            <template v-for="(sub_item, sub_index) in tableColumnList">-->
<!--                                <td :key="sub_index">{{ getTargetData(item.gpData, sub_item, '-') }}</td>-->
<!--                            </template>-->
<!--                            <td>-->
<!--                                <span class="link" @click="editItem(item)">编辑</span>-->
<!--                                |-->
<!--                                <span class="link danger" @click="deleteItem(item)">删除</span>-->
<!--                            </td>-->
<!--                        </tr>-->
<!--                    </template>-->
<!--                </table>-->
                <a-table size="middle" :columns="chartsColumns" rowKey="id" :dataSource="tableDataSource" bordered :pagination="false">
                    <div slot="filterDropdown"
                        slot-scope="{setSelectedKeys, selectedKeys, confirm, clearFilters, column }"
                        style="padding: 8px">
<!--                        {{setSelectedKeys}}-->
<!--                        {{selectedKeys}}-->
<!--                        {{confirm}}-->
<!--                        {{clearFilters}}-->
<!--                        {{column.title}}-->
<!--                        <input title="是否在ECharts中显示此列数据" type="checkbox"-->
<!--                               :checked="isCheckedColumn(column)"-->
<!--                               @change="columnCheckedChange($event, column)"/>-->
                        是否在ECharts中显示此列数据：
                        <a-checkbox :checked="isCheckedColumn(column.title)" @change="columnCheckedChange($event, column.title)"
                                    title="是否在ECharts中显示此列数据" />
                    </div>
                    <a-icon
                        slot="filterIcon"
                        slot-scope="filtered"
                        type="search"
                        :style="{ color: filtered ? '#108ee9' : undefined }"
                    />
                    <template slot="time" slot-scope="text, record">
                        <a-checkbox :checked="isCheckedDataSource(record)" @change="dataSourceCheckedChange($event, record)"
                                    title="是否在ECharts中显示行数据" />
                        {{text}}
                    </template>
                    <template slot="action" slot-scope="text, records, index">
                        <a @click="editItem(records)"><a-icon type="form" /> 编辑</a>
                        <a-divider type="vertical" :key="'record'+index" :text="text"/>
                        <a-popconfirm title="你要删除这行记录吗？" @confirm="deleteItem(records)">
                            <a style="color: #ff0000"><a-icon type="delete" /> 删除</a>
                        </a-popconfirm>
                    </template>
                </a-table>
                {{uncheckedColumn}}
                <a-button style="display: block; margin: 20px auto;" type="primary" icon="plus">添加</a-button>
            </div>
            <div class="chart_box" id="chartBox">
                <template v-for="(item, index) in tableColumnListFiltered">
                    <line-charts :key="index" :index="index" class="charts_item" :title="item"
                                 :x-data="allDate" :y-data="getDataListByKey(item)" />
                </template>
            </div>
        </div>

        <div class="page_loader_box" v-if="pageIsLoading">
            <div class="page_loader">
                <div class="loader-inner line-spin-fade-loader">
                    <div></div>
                    <div></div>
                    <div></div>
                    <div></div>
                    <div></div>
                    <div></div>
                    <div></div>
                    <div></div>
                </div>
                <div class="text">{{ pageLoadingText }}</div>
            </div>
        </div>
    </div>
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
            baseUrl: "//139.224.41.178:8073/wealth/apcount",
            pageIsLoading: true,
            pageLoadingText: "页面加载中",

            metaDataList: [],
            currentEditItemMetaData: '{\n' +
                '    "summation": "1000",\n' +
                '    "sumStandard": "50",\n' +
                '    "gpTime": "2021-12-31",\n' +
                '    "gpData": [\n' +
                '        {\n' +
                '            "name": "涨停1",\n' +
                '            "weightNumber": "0",\n' +
                '            "riseStop": "80",\n' +
                '            "maxNumber": "24",\n' +
                '            "minNumber": "100"\n' +
                '        }\n' +
                '    ]\n' +
                '}',

            timer: false,
            screenWidth: document.body.clientWidth,
            chartBox: null,

            darkMode: false,

            unCheckedData: [],
            uncheckedColumn: [],

        }
    },

    computed: {
        // currentEditItem: {
        //     get: function () {
        //         let item = "";
        //         console.log(this.currentEditItemMetaData);
        //         try {
        //             item = JSON.parse(this.currentEditItemMetaData);
        //             item = JSON.stringify(item, null, 4);
        //         } catch {
        //             item = this.currentEditItemMetaData;
        //         }
        //
        //         return item;
        //     },
        //     set: function (e) {
        //         this.currentEditItemMetaData = e;
        //     }
        // },

        tableDataSource: function () {
            let list = [], itemObject = {};
            for (let i = 0; i < this.metaDataList.length; i++) {
                let item = {};
                try {
                    item = JSON.parse(this.metaDataList[i].gpData);
                    itemObject = this.arrayToObject(item)
                } catch (err){
                    console.error(err);
                }
                list.push({
                    id: this.metaDataList[i].id,
                    gpTime: this.metaDataList[i].gpTime,
                    gpDataObject: itemObject,
                    gpData: item,
                    metaData: this.metaDataList[i]});
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

        tableColumnListFiltered:function (){
            let list = [];
            for(let i=0; i<this.tableColumnList.length; i++){
                let index = this.uncheckedColumn.findIndex((item) => {
                    return item === this.tableColumnList[i];
                })
                if (index === -1) {
                    list.push(this.tableColumnList[i]);
                }
            }
            return list;
        },

        chartsColumns: function (){
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
            for(let i=0;i<this.tableColumnList.length; i++){
                columnList.push({
                    title: this.tableColumnList[i],
                    dataIndex: "gpDataObject."+this.tableColumnList[i]+".data",
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
                if(this.isCheckedDataSource(this.metaDataList[i])){
                    list.push(this.metaDataList[i].gpTime);
                }
            }
            return list
        },

        chartsSeriesList: function () {
            let list = [];
            for (let i = 0; i < this.tableColumnList.length; i++) {
                if(this.isCheckedColumn(this.tableColumnList[i])){
                    list.push({
                        type: 'line',
                        name: this.tableColumnList[i],
                        data: this.getDataListByKey(this.tableColumnList[i])
                    })
                }
            }

            return list;
        }
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
            this.$axios.get(this.baseUrl + "/list", {params: queryParam}).then((result) => {
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
            for (let i = 0; i < this.tableDataSource.length; i++) {
                let dataList = this.tableDataSource[i].gpData;
                let value = this.getTargetData(dataList, key, 0);
                list.push(value);
            }

            return list;
        },

        // 数组转对象
        arrayToObject: function (array){
            let object = {};
            for(let i=0;i<array.length; i++){
                object[array[i].name] = array[i]
            }
            return object;
        },


        // 编辑一项
        editItem: function (item) {
            let _item = JSON.parse(JSON.stringify(item));
            console.log(item);
            _item.metaData.gpData = JSON.parse(_item.metaData.gpData);
            this.currentEditItemMetaData = JSON.stringify(_item.metaData, null, 4);
        },

        // 为一项添加列
        addFieldItem: function () {
            let currentEditItemMetaData;
            let item = {
                "weightNumber": "80",
                "minNumber": "80",
                "riseStop": "80",
                "maxNumber": "80",
                "name": "新字段"
            };
            try {
                currentEditItemMetaData = JSON.parse(this.currentEditItemMetaData);
                currentEditItemMetaData.gpData.push(item)
                currentEditItemMetaData = JSON.stringify(currentEditItemMetaData, null, 4);
            } catch (err) {
                console.error("解析失败。", err)
                alert("当前数据不是正确的JSON，不能追加列数据");
                currentEditItemMetaData = this.currentEditItemMetaData;
                // gpData = this.currentEditItemMetaData.gpData + JSON.stringify(item, null, 4);
            }
            this.currentEditItemMetaData = currentEditItemMetaData;
        },

        // 取消编辑项目
        cancelEditItem: function () {
            this.currentEditItemMetaData = '{\n' +
                '    "summation": "1000",\n' +
                '    "sumStandard": "50",\n' +
                '    "gpTime": "2021-12-31",\n' +
                '    "gpData": [\n' +
                '        {\n' +
                '            "name": "涨停1",\n' +
                '            "weightNumber": "0",\n' +
                '            "riseStop": "80",\n' +
                '            "maxNumber": "24",\n' +
                '            "minNumber": "100"\n' +
                '        }\n' +
                '    ]\n' +
                '}'
        },

        // 保存编辑项目
        saveEditItem: function () {
            let _this = this;
            //检查元数据的值是不是正确的JSON。如果不是，拒绝保存
            let currentEditItemMetaData;
            try {
                currentEditItemMetaData = JSON.parse(this.currentEditItemMetaData);
                // currentEditItemMetaData.gpData = JSON.stringify(currentEditItemMetaData.gpData);
            } catch (err) {
                alert("当前数据不是正确的JSON，不能进行保存操作");
                return false;
            }

            if (currentEditItemMetaData.createTime) {
                delete currentEditItemMetaData.createTime;
            }
            if (currentEditItemMetaData.updateTime) {
                delete currentEditItemMetaData.updateTime;
            }
            console.log(currentEditItemMetaData)

            //调用接口保存
            this.pageIsLoading = true;
            this.$axios.post(this.baseUrl + "/add", currentEditItemMetaData).then((res) => {
                _this.pageIsLoading = false;
                console.log(res);
                if (res.data.success) {
                    alert("保存成功");
                    _this.cancelEditItem();
                    _this.getServerData();
                } else {
                    alert("保存失败。"+res.data.message);
                }
            }).catch((err) => {
                _this.pageIsLoading = false;
                alert("保存错误。"+err.toString());
                console.error(err);
            })
        },

        // 渲染底部折线图表


        // 切换日间、夜间模式
        changeThemeColor: function (){
            const domTarget = document.querySelector('#app');

            if(this.darkMode){
                domTarget.style.setProperty('--background_color', '#ffffff');
                domTarget.style.setProperty('--text_color', '#000000');
            }
            else{
                domTarget.style.setProperty('--background_color', '#000000');
                domTarget.style.setProperty('--text_color', '#59f6fb');
            }

            this.darkMode = !this.darkMode;

        },

        // 检查此字段是否选中
        isCheckedColumn: function (item){
            let index = this.uncheckedColumn.findIndex((sItem) => {return sItem === item})
            return index === -1
        },

        // 此字段选中变化
        columnCheckedChange: function (e, item){
            console.log(e, item);
            let index = this.uncheckedColumn.findIndex((sItem) => {return sItem === item})
            if(index > -1){
                // 将此字段从unChecked 列表移除
                this.uncheckedColumn.splice(index, 1);
            }
            else{
                // 将字段追加到 unChecked 列表
                this.uncheckedColumn.push(item);
            }

        },

        // 此行数据是否选中
        isCheckedDataSource: function (item){
            console.log(item);
            let index = this.unCheckedData.findIndex((sItem) => {return sItem === item.id})
            return index === -1;
        },

        // 此行数据选中变化
        dataSourceCheckedChange: function (e, item){
            console.log(item);
            item = JSON.parse(JSON.stringify(item));
            item = item.metaData;
            let index = this.unCheckedData.findIndex((sItem) => {return sItem === item.id})
            if(index > -1){
                // 将此字段从unChecked 列表移除
                this.unCheckedData.splice(index, 1);
            }
            else{
                // 将字段追加到 unChecked 列表
                this.unCheckedData.push(item.id);
            }

        },

        // 删除一项
        deleteItem: function (item){
            let _this = this;
            if (confirm("您是否删除记录"+item.metaData.gpTime+"("+item.metaData.id+")")){
                this.$axios.delete(this.baseUrl+"/delete", {params: {id: item.metaData.id}}).then((res) => {
                    if(res.data.success){
                        _this.$message.success("删除成功");
                        _this.getServerData()
                    }
                    else{
                        _this.$message.warn("删除失败。"+res.data.message);
                    }
                }).catch((err) => {
                    _this.$message.error("删除错误"+err.toString());
                })
            }
        },
    },

    created: function () {
        this.getServerData()
    },
}
</script>

<style scoped lang="less" >
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

            .button{
                flex: auto;
                height: 50px;
                display: flex;
                align-items: center;
                justify-content: center;
                cursor: pointer;
                user-select: none;

                &.add{
                    width: 100%;
                    border-bottom: 2px solid #000000;
                    transition: border 200ms ease-in-out;
                }

                &.cancel{
                    border-right: 2px solid #000000;
                    transition: border 200ms ease-in-out;
                }

                &.dark_mode{
                    width: 100%;
                    border-top: 2px solid #000000;
                    transition: border 200ms ease-in-out;
                }

                &:hover{
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

        .chart_box{
            width: 100%;
            height: calc(100vh - 20px - 300px);
            overflow: auto;
            display: flex;
            flex-wrap: wrap;
            align-content: flex-start;

            .charts_item{
                width: 25%;
                height: 250px;
            }
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
