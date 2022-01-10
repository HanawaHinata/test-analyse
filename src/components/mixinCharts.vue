<template>
    <div class="mixin_charts_box" :id="'chartBox'+index"></div>
</template>

<script>
let echarts = require('echarts/lib/echarts')
// 引入柱状图组件
require('echarts/lib/chart/line');
// 引入柱状图组件
require('echarts/lib/chart/bar');
// 引入提示框和title组件
require('echarts/lib/component/tooltip')
require('echarts/lib/component/title')
require('echarts/lib/component/grid')

export default {
    name: "mixinCharts",
    props: {
        index: {
            type: [Number, String],
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
        series: {
            type: Array,
            default: function () {
                return [];
            }
        },
        type: {
            type: String,
            require: true,
            default: "line",
        },
        xAxisPosition: {
            type: String,
            require: true,
            default: "top",
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
                    type: 'category', boundaryGap: (this.type==='bar'),
                    axisLabel: {color: (this.darkMode ? '#00E8E9' : '#000000'), fontSize: 13},
                    axisLine: {show: true, lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035')}},
                    axisTick: {show: false,},
                    splitLine: {show: true, lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035')}},
                    data: this.xData,
                    nameLocation: 'end',
                    position: this.xAxisPosition,
                }],
                yAxis: [{
                    type: 'value', name: '值',
                    nameTextStyle: {color: (this.darkMode ? '#00E8E9' : '#000000'), fontSize: 13},
                    axisLabel: {formatter: '{value}', color: (this.darkMode ? '#00E8E9' : '#000000'), fontSize: 13},
                    axisLine: {lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035'),}},
                    axisTick: {show: false},
                    splitLine: {show: true, lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035')}},
                    inverse: this.xAxisPosition==='top'
                }],
                series: this.series
            };
            console.log("渲染图表", chartBoxOption);
            _this.chartBox.setOption(chartBoxOption, true);
        },
    },

    mounted: function () {
        let _this = this;
        // window.addEventListener("resize", function () {
        //     window.screenWidth = document.body.clientWidth;
        //     _this.screenWidth = window.screenWidth;
        // })

        let domTarget = document.getElementById('chartBox' + this.index);
        domTarget.addEventListener("resize", function () {
            _this.screenWidth = domTarget.screenWidth;
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
        series: function (e) {
            console.log("series变化",e)
            this.setChart()
        },
        type: function (e) {
            console.log("type变化",e)
            this.setChart()
        },
        xAxisPosition: function (e) {
            console.log("xAxisPosition变化",e)
            this.setChart()
        },
    }
}
</script>

<style scoped>
.mixin_charts_box {
    width: 100%;
    height: 100%;
    overflow: hidden;
}
</style>
