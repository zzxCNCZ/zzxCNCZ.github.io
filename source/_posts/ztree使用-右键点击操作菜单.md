---
title: ztree使用(右键点击操作菜单)
date: 2017-11-23 09:53:11
categories:
- Javascript
tags:
- ztree
---

## Ztree的使用
* 效果展示
![文件夹操作](http://ozrd9k2sg.bkt.clouddn.com/img/%E6%96%87%E4%BB%B6%E5%A4%B9%E6%93%8D%E4%BD%9C.png)
![文件操作](http://ozrd9k2sg.bkt.clouddn.com/img/%E6%96%87%E4%BB%B6%E6%93%8D%E4%BD%9C.png)
* html
```javascript
  <ul id="fileTree" class="ztre" style="background-color:#fff;border:none;margin-top:0;width:100%;height:180px;"></ul>
```
> 引入`css`文件ztree/css/metroStyle/metroStyle.css  ztree/js/jquery.ztree.core-3.5.js

* js
1. 配置`setting`,[ztreeapi](http://www.treejs.cn/v3/main.php#_zTreeInfo)
```javascript

var zTree, rMenu, imei;  //静态参数，之后用到
  var setting = {
    async: {    //异步加载
        enable: true,
        url: "message/operatefile",  //后台请求地址
        type: "post",
        autoParam: ["id", "level", "name", "pId"], //默认参数，详见api
        otherParam: ["imei", $("#imei").val()], //自定义参数
        dataFilter: filter
    },
    data: {
        simpleData: {
            enable: true,
            idKey: "id",           //自定义配置数据
            pIdKey: "pId",
            rootPId: 2
        }
    },
    view: {
        expandSpeed: "",
        dblClickExpand: false

    },
    callback: {
        onRightClick: OnRightClick,          //返回操作，右击树点击事件
    }
};
```
<!--more-->

2. 初始化架子啊
```javascript
function refreshFileInfo() {
    $.fn.zTree.init($("#fileTree"), setting);   //初始化
    zTree = $.fn.zTree.getZTreeObj("fileTree");
    rMenu = $("#rMenu");  //右击控件
}
```
* 右击控件
```javascript
 <style type="text/css">
        div#rMenu {
            position: absolute;
            visibility: hidden;
            top: 0;
            background-color: #f9f9f9;
            text-align: left;
            padding: 2px;
        }

        div#rMenu a {
            cursor: pointer;
            list-style: none outside none;
        }

    </style>

<div id="rMenu">
        <a href="javascript:void(0);" id="r1" class="list-group-item" onclick="Createfloder();">创建文件夹</a>   //穿件文件夹方法
        <a href="javascript:void(0);" id="r2" class="list-group-item" onclick="Deletefile();">删除文件</a>
        <a href="javascript:void(0);" id="r3" class="list-group-item" onclick="Renamefile();">重命名文件</a>
        <a href="javascript:void(0);" id="r4" class="list-group-item" onclick="Downloadfile();">下载文件</a>
        <a href="javascript:void(0);" id="r5" class="list-group-item" onclick="Uploadfile();">上传文件</a>
    </div>
```
3. 右击事件js
```javascript
// 在ztree上的右击事件
function OnRightClick(event, treeId, treeNode) {
    if (!treeNode && event.target.tagName.toLowerCase() != "button" && $(event.target).parents("a").length == 0) {
        showRMenu("root", event.clientX, event.clientY, treeNode);
    } else if (treeNode && !treeNode.noR) {
        showRMenu("node", event.clientX, event.clientY, treeNode);
    }
}

//显示右键菜单
function showRMenu(type, x, y, treeNode) {
    if (treeNode.name.indexOf(".") > 0) {
        $("#rMenu ul").show();
        $("#r1").hide();

        $("#r5").hide();
        $("#r2").show();
        $("#r3").show();
        $("#r4").show();
    } else {
        $("#rMenu ul").show();
        $("#r1").show();

        $("#r5").show();
        $("#r2").hide();
        $("#r3").hide();
        $("#r4").hide();
    }

    y += $(window).scrollTop();
    x += $(window).scrollLeft();
    //alert($(window).scrollTop()+"ttt"+$(window).scrollLeft()+"lll"+x+"xxxx"+y);
    rMenu.css({"top": y + "px", "left": x + "px", "visibility": "visible"}); //设置右键菜单的位置、可见
    $("body").bind("mousedown", onBodyMouseDown);
}
//隐藏右键菜单
function hideRMenu() {
    if (rMenu) rMenu.css({"visibility": "hidden"}); //设置右键菜单不可见
    $("body").unbind("mousedown", onBodyMouseDown);
}
//鼠标按下事件
function onBodyMouseDown(event) {
    if (!(event.target.id == "rMenu" || $(event.target).parents("#rMenu").length > 0)) {
        rMenu.css({"visibility": "hidden"});
    }
}
```

4. 举列创建文件夹方法
```javascript
//创建文件夹
function Createfloder() {
    hideRMenu();
    var selectNodes = zTree.getSelectedNodes();
    //alert(selectNodes[0].name+"11"+selectNodes[0].id+"22");
    // alert(typeof(selectNodes)+""+selectNodes[0]+""+selectNodes.length);
    if (selectNodes.length == '1') {
        fileDialogInit('新建文件夹', '文件夹名', newfile, selectNodes);   //显示模态框，并返回操作
        $("#fileDialog .modal-body .target").parent().parent().addClass("hidden");
    } else {
        toastr.info('请点击选择文件');
    }
}


//文件操作模态框
function fileDialogInit(action, notice, callback, selectNodes) {
    $('#fileDialog .btn-success').unbind("click");
    $("#fileDialog .btn-success").click(function () {
        if (callback) {
            callback(selectNodes);
        }
        $("#fileDialog").modal('hide');
    });
    $("#fileDialog .modal-body .target").parent().parent().removeClass("hidden");
    $("#name").parent().parent().removeClass("hidden");
    var path = selectNodes[0].id;
    var target = selectNodes[0].name;
    $("#fileDialog .modal-title").html(action);
    $("#fileDialog #name").val('');
    $("#fileDialog .modal-body .path").html('当前路径：' + path);
    $("#fileDialog .modal-body .notice").html(notice);
    $("#fileDialog .modal-body .target").html('当前文件：' + target);
    $("#fileDialog").modal("show");
}



 <!-- 文件操作模态框HTML -->
    <div class="modal fade in" id="fileDialog" tabindex="-1" role="dialog">
        <div class="modal-dialog" role="document">
            <div class="modal-content">
                <div class="modal-header" id="norHeader">
                    <button type="button" class="close" data-dismiss="modal" aria-label="Close"><span
                            aria-hidden="true">&times;</span></button>
                    <h4 class="modal-title"></h4>
                </div>
                <div class="modal-body">
                    <div class="form-horizontal">
                        <div class="form-group">
                            <div class="ol-xs-12 col-sm-12 col-md-12 col-lg-12">
                                <label class="path control-label"></label>
                            </div>
                        </div>
                        <div class="form-group">
                            <div class="ol-xs-12 col-sm-12 col-md-12 col-lg-12">
                                <label class="target control-label"></label>
                            </div>
                        </div>
                        <div class="form-group">
                            <div class="ol-xs-2 col-sm-2 col-md-2 col-lg-2">
                                <label class="control-label notice"></label>
                            </div>
                            <div class="col-xs-8 col-sm-8 col-md-8 col-lg-8">
                                <input type="text" id="name" class="form-control"/>
                            </div>
                        </div>
                    </div>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-gray" data-dismiss="modal">取消</button>
                    <button type="button" class="btn btn-success">确定</button>
                </div>
            </div>
        </div>
    </div>



//新建文件操作
function newfile(selectNodes) {
    var name = $("#name").val();
    //alert(selectNodes+name);
    if (name == null || name == "") {
        toastr.warning('文件名不能为空!');
        return;
    }
    $.ajax({
        type: "post",
        url: "message/createfloder",
        data: {
            path: selectNodes[0].id,
            name: name,
            imei: imei
        },
        success: function (result) {
            if (result.msg == "ok") {
                toastr.success('创建文件成功!');
                refreshFileInfo();
            } else {
                toastr.error('创建文件失败!');
            }
        },
        error: function () {
            toastr.error('连接出错!');
        }
    });

}
```

## java后台

```javascript
public List<NodeBean> queryTreeData(JSONObject ret,String treepid,String treename) {
        List<NodeBean> nodeBeanList= new ArrayList<NodeBean>();
        JSONArray retdata=ret.getJSONArray("data");    //获取查询参数
        for(int i=0;i<retdata.size();i++){
            NodeBean treedata=new NodeBean();
            treedata.setId(treepid+"/"+retdata.getJSONObject(i).getString("name"));   //拼装数据
            treedata.setpId(treepid+treename+"/");
                /*treedata.setpId(String.valueOf(i));*/
            treedata.setName(retdata.getJSONObject(i).getString("name"));
            if(retdata.getJSONObject(i).getString("type").equals("D")){
                treedata.setIsParent(true);
            }else {
                treedata.setIsParent(false);

            }
            nodeBeanList.add(treedata);
        }
        return nodeBeanList;
    }
```





