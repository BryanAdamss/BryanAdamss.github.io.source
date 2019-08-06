---
title: Bootstrap-table_API
tags:
  - 表格
categories:
  - 前端
date: 2017-09-07 20:44:29
---

> 最近在项目中需要使用到表格控件，调研几个常用的表格控件(jquery-dataTable、list.js、jqGrid、Bootstrap-table)后，决定使用Bootstrap-table，特意将常用API记录下来，以备后用。
> 源码可在这https://github.com/BryanAdamss/SourceSave/tree/master/Practice

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <!-- 引入bootstrap样式 -->
    <link rel="stylesheet" href="./vendors/bootstrap-3.3.7-dist/css/bootstrap.min.css">
    <!-- 引入bootstrap-table样式 -->
    <link rel="stylesheet" href="./vendors/bootstrap-table/bootstrap-table.css">
</head>

<body>
    <div class="btn-group hidden-xs" id="js_caremaTableToolBar" role="group">
        <a href="#" class="btn btn-outline btn-default"><i class="glyphicon glyphicon-plus" aria-hidden="true"></i></a>
    </div>
    <table id="js_caremaTable" data-height="400" data-icon-size="outline" data-striped="true">
    </table>
    <script src="./js/jquery.min.js"></script>
    <!-- 引入bootstrap.js -->
    <script src="./vendors/bootstrap-3.3.7-dist/js/bootstrap.min.js"></script>
    <!-- bootstrap-table主js -->
    <script src="./vendors/bootstrap-table/bootstrap-table.js"></script>
    <!-- bootstrap-table本地化文件 -->
    <script src="./vendors/bootstrap-table/locale/bootstrap-table-zh-CN.js"></script>
    <script>
    var caremaTable = $('#js_caremaTable').bootstrapTable({
        url: './json/bootstrap_table_test.json',
        method: 'get', // 请求方式
        uniqueId: 'id', // 每一行的唯一id，一般设置为主键列，此处用数据中的'id'做为每一行的唯一标识；可用在一些方法中如removeByUniqueId，不设置uniqueId时，调用此类方法会出错
        search: true, // 启用搜索
        sortable: true, // 全局配置，是否启用列排序；若为false，即使列上设置了sortable:true，列也无法进行排序
        pagination: true, // 启用分页
        sidePagination: 'client', // 设置在哪里进行分页，可选值为 'client' 或者 'server'。设置 'server'时，必须设置 服务器数据地址（url）或者重写ajax方法
        showRefresh: true, // 启用刷新
        showColumns: true, // 启用内容列下拉框
        showToggle: true, // 是否显示 切换试图（table/card）按钮
        showPaginationSwitch: true, // 是否显示切换分页按钮
        toolbar: '#js_caremaTableToolBar', // 工具栏
        cache: true, // 设置为 false 禁用 AJAX 数据缓存
        singleSelect: false, // 设置True 将禁止radio、checkbox多选，并隐藏选择全部按钮
        class: 'cgh_dfjakdjfklasjdfklajslkdfjlkaf',
        columns: [{
                checkbox: true, // 此列为checkbox

            },
            // {
            //     radio: true // 此列为radio
            // },
            {
                field: 'id',
                title: '编号',
                sortable: true, // 此列表头点击，可进行排序，前提必须是表格的sortable为true
                titleTooltip: '点击可进行排序', // 悬停tooltip
                width: '100px'
            }, {
                field: 'name',
                title: '名称',
            }, {
                field: 'place',
                title: '预置位'
            }, {
                field: 'userName',
                title: '连接用户名'
            }, {
                field: 'password',
                title: '连接密码'
            }, {
                field: 'channel',
                title: '远程频道'
            }, {
                field: 'lng',
                title: '经度'
            }, {
                field: 'lat',
                title: '纬度',
            }, {
                title: '操作',
                formatter: function(value, row, index) {
                    return '<a href="javascript:;" class="text-danger m-l js_delete">删除</a>'
                },
                events: { // 按钮点击事件
                    'click .js_delete': function(e, value, row, index) {
                        console.log(e, value, row, index);
                        if (confirm('确定删除此行吗？')) {
                            caremaTable.bootstrapTable('removeByUniqueId', row.id); // 当表格配置了uniqueId: 'id'时，可通过removeByUniqueId来删除当前行

                            // caremaTable.bootstrapTable('remove', { // 删除name列值为'测试0'的行
                            //     field: 'name',
                            //     values: ['测试0'] // 注意values必须是一个数组
                            // });

                            // caremaTable.bootstrapTable('remove', { // 删除当前行，由于id在创建数据时是唯一的，所以通过点击获取row中的id数据，然后删除id列值为row.id的行；不过还是建议通过配置uniqueId然后通过removeByUniqueId方法来删除
                            //     field: 'id',
                            //     values: [row.id]
                            // });

                        }
                    }
                }
            }
        ],
        onCheck: function(row) { // 单独选中某一个check时触发
            console.log('onCheck', row);
        },
        onUncheck: function(row) {
            console.log('onUncheck', row); // uncheck某一个check时触发
        },
        onCheckAll: function(rows) {
            console.log('onCheckAll', rows); // 全选check时触发
        },
        onUncheckAll: function(rows) {
            console.log('onUncheckAll', rows); // uncheck所有check时触发
        }
    });

    // 常用方法
    // 调用方法的语法：$('#table').bootstrapTable('method', parameter);
    // 全部方法请参阅http://bootstrap-table.wenzhixin.net.cn/zh-cn/documentation/#方法
    // caremaTable.bootstrapTable('getSelections'); // 返回所选的行，当没有选择任何行的时候返回一个空数组。
    // caremaTable.bootstrapTable('getAllSelections'); // 返回所有选择的行，包括搜索过滤前的，当没有选择任何行的时候返回一个空数组。
    // caremaTable.bootstrapTable('getData'); // 获取当前加载的数据。
    // caremaTable.bootstrapTable('getRowByUniqueId', 3); // 根据 uniqueId 获取行数据。
    // caremaTable.bootstrapTable('load', data); // 加载数据到表格中，旧数据会被替换。
    // caremaTable.bootstrapTable('showAllColumns'); // 显示所有列。
    // caremaTable.bootstrapTable('hideAllColumns'); // 隐藏所有列。
    // caremaTable.bootstrapTable('append'); // 添加数据到表格在现有数据之后。
    // caremaTable.bootstrapTable('prepend'); // 插入数据到表格在现有数据之前。
    // caremaTable.bootstrapTable('remove', { field: 'id', values: someArr }); // 从表格中删除数据，包括两个参数： field: 需要删除的行的 field 名称。values: 需要删除的行的值，类型为数组。
    // caremaTable.bootstrapTable('removeAll'); // 删除表格所有数据。
    // caremaTable.bootstrapTable('removeByUniqueId'); // 根据 uniqueId 删除指定的行。
    // caremaTable.bootstrapTable('insertRow', { index: 1, row: row }); // 插入新行，参数包括：index: 要插入的行的 index。row: 行的数据，Object 对象。
    // caremaTable.bootstrapTable('updateRow', { index: 1, row: row }); // 更新指定的行，参数包括：index: 要更新的行的 index。row: 行的数据，Object 对象。
    // caremaTable.bootstrapTable('showRow', { index: 1 }); // 显示指定的行，参数包括：index: 要更新的行的 index 或者 uniqueId。isIdField: 指定 index 是否为 uniqueid。
    // caremaTable.bootstrapTable('hideRow', { index: 1 }); // 隐藏指定的行，参数包括：index: 要更新的行的 index 或者 uniqueId。isIdField: 指定 index 是否为 uniqueid。
    // caremaTable.bootstrapTable('checkAll'); // 选中所有行
    // caremaTable.bootstrapTable('uncheckAll'); // uncheck所有行
    // caremaTable.bootstrapTable('check', 0); // 选中第一行
    // caremaTable.bootstrapTable('uncheck', 0); // 取消选中第一行
    // caremaTable.bootstrapTable('checkBy', { field: "field_name", values: ["value1", "value2", "value3"] }); // 选中field_name为value1、value2、value3的行
    // caremaTable.bootstrapTable('uncheckBy', { field: "field_name", values: ["value1", "value2", "value3"] }); // 取消选中field_name为value1、value2、value3的行

    $(window).resize(function() { // 防止thead和tbody在缩放情况下不对齐
        caremaTable.bootstrapTable('resetView');
    });
    </script>
</body>

</html>
```
