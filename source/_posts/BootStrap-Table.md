---
title: BootStrap-Table
date: 2017-11-23 16:44:31
categories:
- Javascript
- BootStrap
tags:
- BootStrap-Table
---

[Bootstrap中文网](http://wenda.bootcss.com/topic/)
[Bootstrap-table中文api](http://blog.csdn.net/rickiyeat/article/details/56483577)
[Bootstrap-table官网](http://bootstrap-table.wenzhixin.net.cn/zh-cn/documentation/)
## 引入css
> bootstrap/css/bootstrap.min.css
> bootstrap-table/src/bootstrap-table.css
* 行内编辑
[文档地址](http://vitalets.github.io/x-editable/docs.html)
> bootstrap3-editable/css/bootstrap-editable.css
## 引入js
> bootstrap/js/bootstrap.min.js
> bootstrap-table/src/bootstrap-table.js
> bootstrap-table/src/extensions/filter-control/bootstrap-table-filter-control.js
> bootstrap-table/src/extensions/toolbar/bootstrap-table-toolbar.js
> `https://cdn.bootcss.com/x-editable/1.5.1/bootstrap3-editable/js/bootstrap-editable.min.js`
> bootstrap-table/src/extensions/editable/bootstrap-table-editable.js
* 中文包
> bootstrap-table/locale/bootstrap-table-zh-CN.min.js

## Html代码
* 可在html配置，或者js配置(本文html配置)
<!--more-->
```javascript
<table id="phoneList" class="table table-bordered"
            <%-- data-toggle="table"
             data-pagination="true"
             data-side-pagination="server"
             data-page-list="[5, 10, 20, 50, 100, 200]"
             data-show-columns="true"
             data-show-refresh="true"
             data-show-toggle="true"
             data-show-export="true"
             data-toolbar="#toolbar"
             data-query-params="queryParams"
             data-url="asset/queryTerminalListJson"--%>

                   data-advanced-search="true"
                   data-id-table="advancedTable"
                   data-filter-control="true"
                  <%-- data-filter-show-clear="true"--%>
                   data-classes="table table-no-bordered"
                   data-sort-name="mjjh"
                   <%--data-sort-order="desc"--%>
                   data-query-params="queryParams"          
            >

                <thead>
                <tr>
                    <%--<th data-field="state" data-checkbox="true"></th>--%>
                    <th data-field="mjjh" data-sortable="true" data-formatter="mjjhFormatter">民警警号</th>
                    <th data-field="phone_number" <%--data-formatter="phonenumFormatter"--%> data-sortable="true">手机号</th>
                    <th data-field="mjxm" data-sortable="true">民警姓名</th>
                    <th data-field="jgmc" data-sortable="true">机构名称</th>
                    <th data-field="imei" data-sortable="true">IMEI</th>
                    <th data-field="imsi" data-sortable="true" data-visible="false"
                        data-formatter="imsiFormatter"data-events="actionEvents">IMSI</th>
                    <th data-field="cid" data-sortable="true" data-visible="false">CID</th>
                    <th data-field="product" data-sortable="true" data-visible="false">产品名</th>
                    <th data-field="manuFacturer" data-formatter="manuFacturerFormatter" data-visible="false"
                        data-sortable="true"<%--data-filter-control="select"--%>>终端厂商
                    </th>
                    <th data-field="brand" data-sortable="true" data-visible="false">终端品牌
                    </th>
                    <th data-field="online" data-formatter="onlineFormatter" data-align="center" data-events="actionEvents"
                        data-switchable="false">
                        在线
                    </th>
                    <th data-field="action" data-formatter="actionFormatter" data-align="center" data-events="actionEvents"
                        data-switchable="false">
                        远程操作
                    </th>

                </tr>
                </thead>
            </table>
```


* js代码
```javascript
$('#phoneList').bootstrapTable({
        url: 'asset/queryTerminalListJson',
        method: 'get',
        toolbar: '#toolbar',
        striped: true,
        cache: false,
        pagination: true,
        /*queryParams: function (params) {
            debugger;
            params.mjjh=$("#mjjh").val();
            params.mjxm=$("#mjxm").val();
            params.jgbm=$("#jgbm").val();
            params.beginTime=$("#beginTime").val();
            params.endTime=$("#endTime").val();
            return params;
        },*/
        queryParamsType: "limit",
        detailView: false,//父子表
        sidePagination: "server",
        pageSize: 10,
        pageList: [10, 25, 50, 100],
        /*search: true,*/
        showColumns: true,
        showRefresh: true,
        sortable: true,                     //是否启用排序
        sortOrder: "asc",
        minimumCountColumns: 2,
        clickToSelect: true,
       /* detailView:true,
        detailFormatter:function (index, row) {
            var html = [];
            $.each(row, function (key, value) {

                html.push('<p><b>' + key + ':</b> ' + value + '</p>');
            });
            return html.join('');
        },
        */                             //详情操作
        onClickRow: function (row, $element) {
            curRow = row;
        },
        /*columns: [{                                         //js配置column
            checkbox: true
        },
            {
            field: 'phone_number',
            title: '手机号码',
            filter: {
                type: "input"
            }
        }, {
            field: 'mjxm',
            title: '民警警号',
            filter: {
                type: "select",
                data: []
            }
        }, {
            field: 'jgmc',
            title: '机构名称'
        }, {
            field: 'imei',
            title: 'IMEI'/!*,
            formatter: function (value, row, index) {
                return "<a href=\"#\" name=\"imei\" data-type=\"text\" data-pk=\""+row.Id+"\" data-title=\"用户名\">" + value + "</a>";
            }*!/

        },{
            field: 'product',
            title: '型号'
        },{
            field: 'manuFacturer',
            title: '终端厂商',
            sortable: true,
            filterControl:"select",
                editable: {
                    type: 'text',
                    title: '用户名',
                    validate: function (v) {
                        if (!v) return '用户名不能为空';

                    }
                },
            },
         {
            field: 'brand',
            title: '终端品牌'
        },{
            field: 'action',
            title: '操作',
            formatter:actionFormatter
        },
        ],*/
      /*  onLoadSuccess: function (aa, bb, cc) {
            $("#phoneList a").editable();
        },*/

        onLoadSuccess: function (aa, bb, cc) {
            $('#phoneList a').not('.remove').not('.detail-phone').not('.remote_control').not('.online_user').editable({    //行内编辑取消
                url: function (params) {
                    var sName = $(this).attr("name");
                    //curRow[sName] = params.value;
                    curRow.changeType=sName;
                    curRow.changeValue=params.value;
                    $.ajax({
                        type: 'POST',
                        url: "asset/editdevice",
                        data: curRow,
                        dataType: 'JSON',
                        success: function (data, textStatus, jqXHR) {
                            alert('保存成功！');
                        },
                        error: function () { alert("error");}
                    });
                },
                type: 'text'
            });
        },

       /* onEditableSave: function (field, row, oldValue, $el) {
            $.ajax({
                type: "post",
                url: "/Editable/Edit",
                data: { strJson: JSON.stringify(row) },
                success: function (data, status) {
                    if (status == "success") {
                        alert("编辑成功");
                    }
                },
                error: function () {
                    alert("Error");
                },
                complete: function () {

                }

            });
        }*/
    });

```

* js中静默刷新(查询)
```javascript
$("#submit").click(function () {        //提交按钮提交
        $('#phoneList').bootstrapTable('refresh',{
            url: 'asset/queryTerminalListJson',
            silent: true,
        });
    });
```
* 查询参数方法，列参数格式化，绑定方法
```javascript
//查询参数
function queryParams(params)
{
    params.mjjh=$("#mjjh").val();
    params.mjxm=$("#mjxm").val();
    params.jgmc=$("#deptName").val();
    params.phone_number=$("#sjhm").val();
    params.imei=$("#imeinum").val();
    return params;
}

//格式化
function onlineFormatter(value, row, index) {
    if(row.onlinestatus == "1"){
        return [
            '<a class="online_user"  href="javascript:void(0)" title="在线">',
            ' <span class="glyphicon glyphicon-user" style="color: rgb(74,255,73);"></span>',
            '</a>'
        ].join('');
    }else {
        return [
            '<a class="online_user"  href="javascript:void(0)" title="不在线">',
            '<span class="glyphicon glyphicon-user" style="color: rgb(255,3,0);"></span>',
            '</a>'
        ].join('');
    }
}

//绑定的点击操作方法
window.actionEvents = {

}

```

