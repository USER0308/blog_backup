在百度地图 api 的基础上进行改造, 编写高德地图模块.

```
<!doctype html>
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="chrome=1">
    <meta name="viewport" content="initial-scale=1.0, user-scalable=no, width=device-width">
    <style type="text/css">
      body,html,#container{
        height: 100%;
        margin: 0px;
      }
       .panel {
        background-color: #ddf;
        color: #333;
        border: 1px solid silver;
        box-shadow: 3px 4px 3px 0px silver;
        position: absolute;
        top: 10px;
        right: 10px;
        border-radius: 5px;
        overflow: hidden;
        line-height: 20px;
      }
      #input{
        width: 250px;
        height: 25px;
        border: 0;
      }
    </style>
    <title> 快速入门 </title>
  </head>
  <body>
    <div id="container" tabindex="0" style="width:680;height: 1820"></div>
    <div class ='panel'>
     <input id = 'input' value = '输入 point_id 显示位置' onfocus = 'this.value=""'></input>
     <input type="button" onclick="show()" value="show" id="show">
     <div id = 'message'></div>
   </div>
    <script type="text/javascript" src="http://webapi.amap.com/maps?v=1.4.2&key=bdae7574dbeb83c99283dbc69431f613"></script>
    <script type="text/javascript">
        var map = new AMap.Map('container',{
            resizeEnable: true,
            zoom: 16,
            center: [120.168799, 33.354391]
        });
         map.plugin(["AMap.ToolBar"], function() {
            map.addControl(new AMap.ToolBar());
        });
        var my_marks = [];
        var my_points = [{
          'id': '1',
          'position': [120.168799, 33.354391],
        },{
          'id': '5',
          'position': [120.153019, 33.364616],
        },{// 省略部分数据
          'id': '417',
          'position': [120.177977, 33.359098],
        }]
        for (var i = 0; i < my_points.length; i++) {
          var marker = new AMap.Marker({
            position: my_points[i]['position'],//marker 所在的位置
            //label: my_points[i]['id'],
          });
          marker.setLabel({content: my_points[i]['id']})
          my_marks.push(marker);
        }
        for (var i = my_marks.length - 1; i>= 0; i--) {
          my_marks[i].setMap(map);
        }
        //marker.setMap(map);

         input.onchange = function(e){
            var point = input.value;
            var posi = [];
            for (var i = 0; i < my_points.length; i++) {
              if(my_points[i]['id'] === point){
                posi = my_points[i]['position']
              }
            }
            if (posi) {
              map.center = posi
            }else {
              alert('no such point')
            }
        }
        function show(){
          var point = input.value;
          //alert(point)
            var posi = [];
            for (var i = 0; i < my_points.length; i++) {
              if(my_points[i]['id'] === point){
                posi = my_points[i]['position']
                //alert('find')
              }
            }
            if (posi.length!=0) {
              map.setCenter(posi)
              //alert(posi)
            }else {
              alert('no such point')
            }
        }
    </script>
  </body>
</html>
```
