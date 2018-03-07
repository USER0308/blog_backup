    赛题名称：《城市自行车的出行行为分析及效率优化》
    出题单位：零点有数
    赛题背景：城市共享单车体系逐步渗透到各个城市中，给公众出行的 “最后一公里” 带来极大便利。
    任务描述：分析早晚高峰时间段及该城市在高峰时段的自行车大的运动方向，对未来时段（月、周等）进行分站点借还流量预测。
    技术方向：挖掘人的行为特征数据，预测可能抵达的地方（数据挖掘、空间数据运算、路线聚类、可视化表达、最优化设计）
    数据特色：采用了有桩的共享单车数据，降低了随机停放预测的难度。
    商业潜力：对城市规划、城市交通管理有重要参考价值


做 2017 CCF 大数据与人工智能大赛中 <<城市自行车出行行为分析及效率优化>> 题目的时候提供了部分自行车站点的经纬度数据, 需要加以分析, 比如说通过经纬度得知站点所属城市, 获取该城市在这几个月间的天气情况, 进而找出天气对自行车出行的影响.

然后就注册百度地图, 获取开发者 API 接口, 编写自己的地图模块.

```
<!DOCTYPE html>
<html>
<head>
	<meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
	<meta name="viewport" content="initial-scale=1.0, user-scalable=no" />
	<title>check</title>
	<style type="text/css">
		body, html{width: 100%;height: 100%;margin:0;font-family:"微软雅黑";}
		#l-map{height:1000px;width:100%;}
		#r-result{width:100%; font-size:14px;line-height:20px;}
	</style>
	<script type="text/javascript" src="http://api.map.baidu.com/api?v=2.0&ak=AQpjwLXg2Uz35v93TxVCH2K9uMFl8s6D"></script>
</head>
	<script type="text/javascript" src="http://api.map.baidu.com/library/DistanceTool/1.2/src/DistanceTool_min.js"></script>
<body>
	<div id="l-map"></div>
	<div id="r-result">
		<input type="button" value="显示地点" onclick="bdGEO(0)" />
		<input type="button" value="测量" onclick="celiang(0)" />
		<input id="point_id" type="text" style="width:100px; margin-right:10px;" />
		<input type="button" value="定位到该车站 id" onclick="theLocation()" />
		<div id="result"></div>
	</div>
</body>
</html>
<script type="text/javascript">
	// 百度地图 API 功能
	var map = new BMap.Map("l-map");
	map.centerAndZoom(new BMap.Point(120.168799,33.354391), 13);
	map.enableScrollWheelZoom(true);
	var index = 0;
	var myGeo = new BMap.Geocoder();
	var myStructList = [{
		'id': '1',
		'j': 120.168799,
		'w': 33.354391,
	},{
		'id': '5',
		'j': 120.153019,
		'w': 33.364616,
	},{// 省略部分数据
		'id': '417',
		'j': 120.177977,
		'w': 33.359098,
	}
	]
	var points = []
	var marker
	for(var i = 0; i<myStructList.length; i++){
		p = new BMap.Point(myStructList[i]['j'],myStructList[i]['w']),
		points.push(p);
		marker = new BMap.Marker(p);
		map.addOverlay(marker);
		marker.setLabel(new BMap.Label(''+ myStructList[i]['id'],{offset:new BMap.Size(20,-10)}));
	}
	function bdGEO(){
		var pt = points[index];
		geocodeSearch(pt);
		index++;
	}
	function celiang(){
		var myDis = new BMapLib.DistanceTool(map);
		myDis.open();
	}
	function geocodeSearch(pt){
		if(index < points.length-1){
			setTimeout(window.bdGEO,400);
		}
		myGeo.getLocation(pt, function(rs){
			var addComp = rs.addressComponents;
			var zone = 0;
			if(addComp.district === '盐都区'){
				zone = 0;
			}else if(addComp.district === '亭湖区'){
				zone = 1;
			}else {
				zone = -1;
			}
			document.getElementById("result").innerHTML += myStructList[index-1]['id'] + "" + zone +"<br/>";
			// document.getElementById("result").innerHTML += myStructList[index-1]['id'] + "." +points[index-1].lng + "," + points[index-1].lat + "：" + addComp.province + "," + addComp.city + "," + addComp.district + "," + addComp.street + "," + addComp.streetNumber + "<br/><br/>";
		});
	}
	function theLocation(){
		tmp_id = document.getElementById('point_id').value
		for (var i = myStructList.length - 1; i>= 0; i--) {
			if(myStructList[i]['id'] == tmp_id){
				locate = new BMap.Point(myStructList[i]['j'],myStructList[i]['w']);
				map.panTo(locate);
				return;
			}
		};
		alert('No such point');
	}
</script>
```
