---
title: ospService Cals
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

			document.getElementById("result").innerHTML=((100-m)*0.4+(100-z+y)*0.3).toFixed(2);
			// alert(m+","+y+","+z);
		}

		function money(){
			var money=document.getElementById("gongzi").value*1;
			document.getElementById("money").innerHTML=money>800?((money-800)*0.8+800).toFixed(3):money;
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

## Quick Start

### 绩效得分

``` bash
$ hexo new "My New Post"
```

<p style="color: #FBBF05">提醒：%不用输入</p>


<div style="display: flex;">
	<input type="text" placeholder="不满意率%" id="bumanyi" />
	<input type="text" placeholder="一解率%" id="yijie" />
	<input type="text" placeholder="转接率%" id="zhuanjie" />
	<input type="button" value="提交" onclick="calculate()" />
	<p>实际得分：<span id="result"></span></p>
</div>
