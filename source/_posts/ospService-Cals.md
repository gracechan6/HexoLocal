---
title: 淘宝云客服相关小功能使用
date: 2017-09-05 10:21:49
tags:
---

<script>
var _hmt = _hmt || [];
(function() {
  var hm = document.createElement("script");
  hm.src = "https://hm.baidu.com/hm.js?55cd88d2127e1e7e0d8a8e6b5b6a8c59";
  var s = document.getElementsByTagName("script")[0]; 
  s.parentNode.insertBefore(hm, s);
})();
</script>


<script type="text/javascript">
		function calculate(){
			var m=document.getElementById("bumanyi").value*1;
			var y=document.getElementById("yijie").value*1;
			var z=document.getElementById("zhuanjie").value*1;

			document.getElementById("result").innerHTML=((100-m)*0.5+(100-z)*0.3+y*0.2).toFixed(2);
			// alert(m+","+y+","+z);
		}

		function calmoney(){
			var money=document.getElementById("gongzi").value*1;
			var oldGet= money>800?((money-800)*0.8+800).toFixed(3):money ;
			var newGet;
			var mores="";
			if(money<515){
			    newGet=money;
			}else{
			    var a=(money/1.03).toFixed(2);
			    var zengzhi=(a*0.03).toFixed(2);
			    var chengjian=(zengzhi*0.07).toFixed(2);
			    var jiaofu=(zengzhi*0.03).toFixed(2);
			    var difangjiaofu=(zengzhi*0.02).toFixed(2);
			    var geren=(money-zengzhi-chengjian-jiaofu-difangjiaofu);
			    geren=((geren>4000)?geren*0.16:(geren>800?((geren-800)*0.2):0)).toFixed(2);
			    newGet=money-zengzhi-chengjian-jiaofu-difangjiaofu-geren;
			    mores="(增值税："+zengzhi+" 城建税："+chengjian+" 教附税："+jiaofu+" 地方教附税："+difangjiaofu+" 个人所得税"+geren+")";
			}
			document.getElementById("realMoney").innerHTML="旧版税后:"+(oldGet*1).toFixed(2)+"<br/>新版税后:"+(newGet*1).toFixed(2)+mores+"<br/>特云税后："+(newGet*1+(geren*1>500?500:geren*1));
		}

		var manyiAll=0;
		var bumanyiAll=0;
		var yibanAll=0;
		function calEvaluate(){
			var manyidu=document.getElementById("manyidu").value/100;
			yibanAll=document.getElementById("yiban").value*1;
			var bumanyidu=document.getElementById("bumanyidu").value/100;
			if(yibanAll==0){ 
				alert("一般为0，无法计算哦~");
				return;
			}
			var all=Math.round(yibanAll/(1-manyidu-bumanyidu));
			manyiAll=Math.round(all*manyidu);
			bumanyiAll=Math.round(all*bumanyidu);

			document.getElementById("evaluate").innerHTML="总评价个数："+all+" 满意个数："+manyiAll+" 不满意个数："+bumanyiAll;
		}

		function calNewEvaluate(){
			var manyinum=document.getElementById("manyinum").value*1+manyiAll;
			var yibannum=document.getElementById("yibannum").value*1+yibanAll;
			var bumanyinum=document.getElementById("bumanyinum").value*1+bumanyiAll;

			var all=manyinum+yibannum+bumanyinum;
			document.getElementById("predictevaluate").innerHTML=" 满意度："+(manyinum/all).toFixed(4)*100+"% 不满意度："+(bumanyinum/all).toFixed(4)*100+"%";
		}

		var allDutyNum=0;
		var qingjia=0;
		function calAllDutyNum(){
			qingjia=document.getElementById("qingjia").value*1
			var chuqin=document.getElementById("chuqin").value*1;

			allDutyNum=Math.round(qingjia/(1-chuqin/100));
			document.getElementById("allDutyNum").innerHTML=allDutyNum;
		}

		function calNewDutyNum(){
			var newQingjia=document.getElementById("newQingjia").value*1;

			document.getElementById("predictDutyNum").innerHTML=(1-(newQingjia+qingjia)/(allDutyNum+newQingjia)).toFixed(4)*100+"%";
		}

</script>

<style>
/* 用来控制 垂直居中  或者完全居中  start*/
.verMidTabDad{/*控制居中方式一：父节点*/
    display: table;
}
.verMidTabSon{/*控制居中方式一：子节点*/
    display: table-cell;
    vertical-align: middle;
}
.verMidIFDad{/*控制居中方式二.1：父节点*/
    display: inline-flex;
}
.verMidFDad{/*控制居中方式二.2：父节点*/
    display: flex;
}
.verMidIFSon{/*控制居中方式二：子节点*/
    margin-top: auto;
    margin-bottom: auto;
}
.verMidIFSon2{/*控制居中方式二：子节点*/
    margin: auto;
}
/* 用来控制 垂直居中  或者完全居中  end  */

/* 用来控制文字超出部分...  start*/
.wordFixed{
    white-space: nowrap;
    text-overflow: ellipsis;
    overflow: hidden;
}
/* 用来控制文字超出部分...  end  */

.osp_grade input{
    padding-left:5px;
    width:100px;
    height:30px;
    margin-right:10px;
}
</style>

### 绩效得分

<p style="color: #FBBF05">提醒：%不用输入</p>

<div class="verMidFDad osp_grade">
	<input type="text" placeholder="不满意率%" id="bumanyi" class="verMidIFSon"/>
	<input type="text" placeholder="一解率%" id="yijie" class="verMidIFSon"/>
	<input type="text" placeholder="转接率%" id="zhuanjie" class="verMidIFSon"/>
	<input type="button" value="提交" onclick="calculate()" class="verMidIFSon"/>
	<p class="verMidIFSon">实际得分：<span id="result"></span></p>
</div>

### 税后薪资所得
<div class="verMidFDad osp_grade">
    <input type="text" placeholder="工资金额" id="gongzi" class="verMidIFSon"/>
    <input type="button" value="提交" onclick="calmoney()" class="verMidIFSon"/>
    <p class="verMidIFSon"><span id="realMoney"></span></p>
</div>

### 计算明天绩效
<div class="verMidFDad osp_grade">
    <input type="text" placeholder="当前满意度%" id="manyidu"  class="verMidIFSon"/>
    <input type="text" placeholder="当前一般个数" id="yiban"  class="verMidIFSon"/>
    <input type="text" placeholder="当前不满意度%" id="bumanyidu"  class="verMidIFSon"/>
    <input type="button" value="提交" onclick="calEvaluate()"  class="verMidIFSon"/>
    <p  class="verMidIFSon">结论:<span id="evaluate"></span></p>
</div>
<div class="verMidFDad osp_grade">
    <input type="text" placeholder="新增满意个数" id="manyinum"  class="verMidIFSon"/>
    <input type="text" placeholder="新增一般个数" id="yibannum"  class="verMidIFSon"/>
    <input type="text" placeholder="新增不满意个数" id="bumanyinum"  class="verMidIFSon"/>
    <input type="button" value="提交" onclick="calNewEvaluate()"  class="verMidIFSon"/>
    <p  class="verMidIFSon">预计明天绩效：<span id="predictevaluate"></span></p>
</div>

### 出勤率计算
<div class="verMidFDad osp_grade">
    <input type="text" placeholder="请假小时数" id="qingjia"  class="verMidIFSon"/>
    <input type="text" placeholder="当前出勤率%" id="chuqin"  class="verMidIFSon"/>
    <input type="button" value="提交" onclick="calAllDutyNum()"  class="verMidIFSon"/>
    <p  class="verMidIFSon">总班次：<span id="allDutyNum"></span>小时</p>
</div>
<div class="verMidFDad osp_grade">
	<input type="text" placeholder="新增请假小时" id="newQingjia"  class="verMidIFSon"/>
	<input type="button" value="提交" onclick="calNewDutyNum()"  class="verMidIFSon"/>
	<p  class="verMidIFSon">预计明天出勤率：<span id="predictDutyNum"></span></p>
</div>


