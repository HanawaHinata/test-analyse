<template>
    <div class="charts_box">
        <div class="echarts_content" :id="'chartBox'+index" v-resize="resetChartsSize"></div>

    </div>

</template>

<script>
let echarts = require('echarts/lib/echarts')
// 引入折线图组�