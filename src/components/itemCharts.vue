<template>
    <div class="charts_box">
        <div class="echarts_content" :id="'chartBox'+index" v-resize="resetChartsSize"></div>

    </div>

</template>

<script>
let echarts = require('echarts/lib/echarts')
// 引入折线图组件
require('echarts/lib/chart/line');
// 引入柱状图组件
require('echarts/lib/chart/bar');
// 引入提示框和title组件
require('echarts/lib/component/tooltip')
require('echarts/lib/component/title')
require('echarts/lib/component/grid')

export default {
    name: "lineCharts",
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
        yData: {
            type: Array,
            default: function () {
                return [];
            }
        },
        type: {
            type: String,
            default: "line",
        }
    },
    methods: {
        setChart: function () {
            // let _this = this;
            if (!this.chartBox) {
                this.chartBox = echarts.init(document.getElementById('chartBox' + this.index));
            }
            let chartBoxOption = {
                title: {show: true, text: this.title, x: 'center', y: 10, left: '150'},
                grid: {left: 40, right: 40, bottom: 24, top: 50, containLabel: true},
                tooltip: {show: true, trigger: 'axis'},
                xAxis: [{
                    type: 'category', boundaryGap: (this.type === 'bar'),
                    axisLabel: {color: (this.darkMode ? '#00E8E9' : '#8f8f8f'), fontSize: 13},
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
                    // type: 'value', name: '值',
                    nameTextStyle: {color: (this.darkMode ? '#00E8E9' : '#000000'), fontSize: 13},
                    axisLabel: {formatter: '{value}', color: (this.darkMode ? '#00E8E9' : '#8f8f8f'), fontSize: 13},
                    axisLine: {lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035'),}},
                    axisTick: {show: false},
                    splitLine: {show: true, lineStyle: {color: (this.darkMode ? '#ffffff35' : '#00000035')}},
                    min: 'dataMin',
                    // max: 'dataMax',

                }],
                gradientColor: ["#f6efa6", "#d88273", "#bf444c"],
                series: [
                    {
                        type: this.type,
                        name: this.title,
                        data: this.yData,
                        // smooth: true,

                        showSymbol: false,
                        symbol: 'circle',  // 拐点样式
                        symbolSize: 5, // 拐点大小
                        // areaStyle: {
                        //     opacity: 0.5
                        // },
                        itemStyle: {
                            normal: {
                                barBorderRadius: [8, 8, 0, 0],
                            },
                            emphasis: {
                                barBorderRadius: [8, 8, 0, 0],
                            }
                        },
                        lineStyle: {
                            normal: {
                                color: '#64e5fc'
                            }
                        }
                    }
                ]
            };
            // console.log("渲染图表", chartBoxOption);
            this.chartBox.setOption(chartBoxOption, true);
        },

        resetChartsSize: function () {
            if (this.chartBox) {
                this.chartBox.resize();
            }
        }
    },
    directives: {
        resize: {
            bind(el, binding) {
                let width = '', height = '';

                function isReize() {
                    const style = document.defaultView.getComputedStyle(el);
                    if (width !== style.width || height !== style.height) {
                        binding.value();  // 关键
                    }
                    width = style.width;
                    height = style.height;
                }

                el.__vueSetInterval__ = setInterval(isReize, 300);
            },
            unbind(el) {
                clearInterval(el.__vueSetInterval__);
            }
        }
    },
    watch: {
        screenWidth(val) {
            console.log(val);
            if (this.chartBox) {
                this.chartBox.resize()
            }
        },
        xData: function () {
            // console.log("x变化", e)
            this.setChart()
        },
        yData: function () {
            // console.log("y变化", e)
            this.setChart()
        },
        type: function () {
            // console.log("type变化", e)
            this.setChart()
        }
    }
}
</script>

<style lang="less" scoped>
.charts_box {
    width: 100%;
    height: 100%;
    overflow: hidden;

    .echarts_content {
        width: 100%;
        height: 100%;
    }
}
</style>
