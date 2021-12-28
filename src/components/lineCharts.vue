<template>
    <div class="charts_box" :id="'chartBox'+index"></div>
</template>

<script>
let echarts = require('echarts/lib/echarts')
// 引入柱状图组件
require('echarts/lib/chart/line')
// 引入提示框和title组件
require('echarts/lib/component/tooltip')
require('echarts/lib/component/title')
require('echarts/lib/component/grid')

export default {
    name: "lineCharts",
    props: {
        index: {
            type: Number,
            require: true,
            default: 0,
        },
        title: {
            type: String,
            require: true,
            default: "折线图",
        },
        xData: {
            type: Array,
            default: function () {
                return [];
            }
        },
        yData: {
            type: Array,
            default: function () {
                return [];
            }
        }
    },
    methods: {
        setChart: function () {
            let _this = this;
            _this.chartBox = echarts.init(document.getElementById('chartBox' + this.index));
            let chartBoxOption = {
                title: {show: true, text: this.title, x: 'center', y: 10,},
                grid: {left: 40, right: 40, bottom: 24, top: 50, containLabel: true},
                tooltip: {show: true, trigger: 'axis'},
                xAxis: [{
                    type: 'category', boundaryGap: false,
                    axisLabel: {color: (this.darkMode ? '#00E8E9' : '#000000'), fontSize: 10},
                    axisLine: {show: true, lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035')}},
                    axisTick: {show: false,},
                    splitLine: {show: true, lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035')}},
                    data: this.xData
                }, {
                    nameLocation: 'end', position: "bottom",
                    splitLine: {show: false}, axisTick: {show: false},
                    axisLine: {show: false}, axisLabel: {show: false}
                }],
                yAxis: [{
                    type: 'value', name: '值',
                    nameTextStyle: {color: (this.darkMode ? '#00E8E9' : '#000000'), fontSize: 10},
                    axisLabel: {formatter: '{value}', color: (this.darkMode ? '#00E8E9' : '#000000'), fontSize: 10},
                    axisLine: {lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035'),}},
                    axisTick: {show: false},
                    splitLine: {show: true, lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035')}}
                }],
                series: [
                    {
                        type: 'line',
                        name: this.title,
                        data: this.yData
                    }
                ]
            };
            console.log("渲染图表", chartBoxOption);
            _this.chartBox.setOption(chartBoxOption, true);
        },
    },
    mounted: function () {
        let _this = this;
        window.addEventListener("resize", function () {
            window.screenWidth = document.body.clientWidth;
            _this.screenWidth = window.screenWidth;
        })

        // 重新渲染图表
        this.setChart()
    },
    watch: {
        screenWidth(val) {
            console.log(val);
            if (this.chartBox) {
                this.chartBox.resize()
            }
        },
        xData: function (e) {
            console.log("x变化",e)
            this.setChart()
        },
        yData: function (e) {
            console.log("y变化",e)
            this.setChart()
        }
    }
}
</script>

<style scoped>
.chart_box {
    width: 100%;
    height: 100%;
    overflow: hidden;
}
</style>
