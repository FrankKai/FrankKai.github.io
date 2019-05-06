---
title: 异步优先队列$nextTick踩坑
date: 
tags: 
---

![](https://upload-images.jianshu.io/upload_images/2976869-382e7370c7cd6b29.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/512)

我需要做到一个异步获取后端数据，然后再去执行绘制echarts图表的效果。
和以往的直接在view层做类似Mustache渲染，也就是类似{{ message }}这样的渲染，是完全不同的。因为Mushtache是直接渲染，Vue的响应式机制会自动通过viewmodel去做这样的响应式更新。
而我们这里要做的事情，是要做到间接的一种渲染，比如这里通过echarts去做渲染。对于这种间接的渲染，Vue需要通过$nextTick去做，因为这样才能获取到最新的，最新的，最新的数据。

下面是一个实例：

```
created(){
    let projectId = this.$route.params.id;
    retriveChartList(projectId).then((res)=>{
        this.drawChartOption.data = res.data;
        this.drawChart();
    });
    //this.drawChart(); 异步队列优先级的原因，放置在这里获取不到最新数据
}
```
```
methods:{
    drawChart:function(){
        //Vue响应式机制的原因，不使用$nextTick也获取不到最新数据                           
        this.$nextTick(()=>{
            let previewChart = echarts.init(document.getElementById('chart-0'));
            let option = {
                    title: {
                        text: "名字"
                    },
                    tooltip: {
                        trigger: 'item'
                    },
                    series: [{
                        type: this.drawChartOption.type,
                        data: this.drawChartOption.data
                    }],
                    xAxis: {
                        name:'x轴',
                        show: true,
                        data:["foo","bar","baz","fbb"]
                    },
                    yAxis: {
                        name:'y轴',
                        show: true
                    }
            };
            previewChart.setOption(option);
            this.previewCtrl = false;
            window.addEventListener('resize', function (){
                previewChart.resize();
            })
        })
    }
} 
```
drawChart()是一个绘制echarts图表的函数，因为起初是没有数据的，所以需要异步从后端获取到数据，但是数据更新和绘制图表是同时进行的，那么如何才能确保先更新数据再绘制图表呢？

这里就需要用到this.$nextTick()，这个函数可以做到数据更新之后等待Vue更新 DOM，DOM状态更新后进行一些操作，但是我们这里其实跳过了DOM更新这一步，我们直接修改的是data中的数据，看起来是跳过了view或者viewModel这一层，直接修改的是model层，但其实本质上Vue监测到的还是DOM的更新，从而更新数据。

那么其实说明：$nextTick()其实可以更加精确的表示为对virtualDOM的一种数据更新。

另外需要注意的一点是：异步队列优先级方面，$nextTick要高于获取后端数据之类的异步api，因此需要在后端异步api的then方法中，执行$nextTick，then(...)方法之外会导致数据未更新。

https://cn.vuejs.org/v2/guide/reactivity.html#%E5%BC%82%E6%AD%A5%E6%9B%B4%E6%96%B0%E9%98%9F%E5%88%97
https://cn.vuejs.org/v2/api/#vm-nextTick
https://segmentfault.com/q/1010000008497683