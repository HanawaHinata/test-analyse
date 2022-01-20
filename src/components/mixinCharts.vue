<template>
    <div class="mixin_charts_box">
        <div class="echarts_content" :id="'chartBox'+index" v-resize="resetChartsSize"></div>
    </div>
</template>

<script>
let echarts = require('echarts/lib/echarts')
// 引入柱状图组�