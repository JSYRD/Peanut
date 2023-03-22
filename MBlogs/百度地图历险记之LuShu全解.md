# 百度地图历险记之LuShu全解

## 项目简介:

接了个双创项目的前端开发的活，主要用地图来展示一些信息，比如时间地点事件什么的。使用了Antd+react解决方案。

欢迎点个star支持一下：[polan233/siyuan-frontend (github.com)](https://github.com/polan233/siyuan-frontend)

[React-BMapGL文档 (baidu.com)](https://lbsyun.baidu.com/solutions/reactBmapDoc)有React中使用百度地图API的封装，可以直接使用，但很可惜事实上只封装了一小部分，很多功能并没有直接封装，但是是可以通过调用[JspopularGL](http://lbsyun.baidu.com/index.php?title=jspopularGL)来实现的，这样我们的操作就更广一些了。

我们如果想要使用jspopularGL，就需要获得到map对象本身。根据文档，我们可以通过ref来实现：

> ### 获取`map`实例
>
> 如果你在业务中需要操作`map`对象，需要`BMapGL.Map`实例的话，可以通过`<Map>`组件实例的`map`属性访问到它。
>
> ```jsx
> <Map ref={ref => {this.map = ref.map}} />
> ```

但是实测，因为异步加载的原因，有时候会出现undefined或者null问题，所以我们可以通过再封装一层的方式：

```jsx
export default class MyMap extends React.Component {
	constructor(props) {
		super(props);
     ...
     this.created = flase;
     ...
 }
 _initMap(){
     ...
     this.map = this.mapRef.map;
     ...
 }
 componentDidMount() {
		if(!this.created) {
			this._initMap();
         this.created = true;
     }
 }

 render() {
		...
     return(
     	<Map ref={ref => {this.mapRef = ref}}>
         	...
         </Map>
     )
 }
}
```

解决这个问题，这样相当于把`ref.map`的获取放到确保组件已经生成后，就不会产生空问题。同时也可以把Map抽象出成一个`React.Components`，在上层直接使用，简单方便。

> 如果你的上层也需要map对象本身，你可以使用
>
> ```jsx
> render() {
> 	const content=
>         <MyMap
>           className="map"
>           ref={(ref) => {this.map = ref}}
>         />
> }
> ```
>
> 来获取。



## LuShu的引入

为了美观 ~~和狂拽酷炫~~ ，我们使用了React-BMapGL已经封装好的[Arc 2D弧线](https://lbsyun.baidu.com/solutions/reactBmapDoc)来展示路径，效果还不错：

![image-20230323002145652](./%E7%99%BE%E5%BA%A6%E5%9C%B0%E5%9B%BE%E5%8E%86%E9%99%A9%E8%AE%B0%E4%B9%8BLuShu%E5%85%A8%E8%A7%A3.assets/image-20230323002145652.png)

> * (数据是随机mock的，所以地名和坐标都是随机的，不代表任何现实真实含义)
> * 每个点的小图标是自行实现的，不展开叙述，后面博文可能会写

完成Arc部分之后，感觉可以加些更酷的元素进去，于是在[open | 百度地图API SDK (baidu.com)](https://lbsyun.baidu.com/index.php?title=open/jsdemo)里面找到了这个东西：

![image-20230323002501338](./%E7%99%BE%E5%BA%A6%E5%9C%B0%E5%9B%BE%E5%8E%86%E9%99%A9%E8%AE%B0%E4%B9%8BLuShu%E5%85%A8%E8%A7%A3.assets/image-20230323002501338.png)

名字叫路书，看起来很不错，是个动画，可以沿着路径走，并且可以附上HTML元素，在飞机上方展示，于是决定引入。

从示例中找到源码：

https://bj.bcebos.com/v1/mapopen/github/BMapGLLib/Lushu/src/Lushu.min.js

首先遇到的第一个问题就是，不知道怎么用。第一次遇到这样的结构：

```js
(function (){
	//function body
})()
```

实际上就是说定义了一个函数并且直接执行之。只需要把它下载下来放到工作文件夹中，重命名一下，然后在我们需要使用的js文件中添加：

```js
import "./Lushu"
```

就可以了，然后就会发现一堆报错。

![image-20230323003643471](./%E7%99%BE%E5%BA%A6%E5%9C%B0%E5%9B%BE%E5%8E%86%E9%99%A9%E8%AE%B0%E4%B9%8BLuShu%E5%85%A8%E8%A7%A3.assets/image-20230323003643471.png)

实际上报错的原因大概只有

* 找不到BMapGL.*

  > 针对这类，直接改成window.BMapGL.*就可以了

* 缺少某个变量

  > 这个文件由于没有直接访问到BMapGL，所以在这些变量使用前定义一下就好，例如：
  >
  > ![image-20230323003726176](./%E7%99%BE%E5%BA%A6%E5%9C%B0%E5%9B%BE%E5%8E%86%E9%99%A9%E8%AE%B0%E4%B9%8BLuShu%E5%85%A8%E8%A7%A3.assets/image-20230323003726176.png)
  >
  > 这两个变量前面加上const或者var就可以了，声明一下。

* defaultIcon没有

  > 我们可以手动定义一个，比如像示例里一样用base64编码的图片：
  >
  > ```js
  >       var defaultIcon = "data:image/png;base64,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";
  >       //如果不是默认实例，则使用默认的icon
  >       if (!(this._opts.icon instanceof window.BMapGL.Icon)) {
  >         this._opts.icon = defaultIcon;
  >       }
  > ```
  >
  > 这么长的这一段就是用base64编码的一个图片，当然也可以找个png转base64编码的工具自己魔改。



目前为止终于能用了。由于一开始比较懒并不想去分析源码，就选择去搜索了现有的关于lushu的使用方法，发现似乎lushu有很多版本。例如上文提到过的版本，和[地图JS API示例 | 百度地图开放平台 (baidu.com)](https://lbsyun.baidu.com/jsdemo.htm#webgl1_7)这里的大地线路书等等。目前为止功能较为全面的是这里的大地线路书中的版本：

[api.map.baidu.com/library/LuShu/gl/src/LuShu_min.js](http://api.map.baidu.com/library/LuShu/gl/src/LuShu_min.js)

不知道是什么原因。

这个版本在基础功能之上实现了`geodesic`,`autoCenter`等功能，比较实用。

部署使用之后发现问题：lushu的运动轨迹和Arc的轨迹并不重合，于是决定开始自行魔改。



## 部分源码解读

首先明确目标：让lushu的运动轨迹和Arc的轨迹重合。所以大概思路是

1. 明确Arc是如何绘制的
2. 明确lushu是如何运动的
3. 修改源码，使得lushu按照Architect的轨迹运动

首先来看Arc源码(node_modules/react-bmapgl/dist/Custom/Arc.js)

![image-20230323005120882](./%E7%99%BE%E5%BA%A6%E5%9C%B0%E5%9B%BE%E5%8E%86%E9%99%A9%E8%AE%B0%E4%B9%8BLuShu%E5%85%A8%E8%A7%A3.assets/image-20230323005120882.png)

重点是`if(this.props.data)`中的部分。可以看到Arc的轨迹实际上是通过构造`OdCurve`，再使用`OdCurve.getPoints()`方法获得轨迹中点的坐标的。

> ### OdCurve
>
> 通过传入2个或2个以上的坐标点，来依次生成od曲线坐标集。
> 该曲线为2D弯曲方式，且不同于大地曲线，大地曲是根据球面最短距离来计算的，距离太近的2个点基本不会弯曲，而这个Od曲线的生成算法不同，即使很短的距离也会弯曲。
>
> OdCurve提供了两个方法：
>
> ### getPoints
>
> `描述`：getPoints({number}|{undefined})
>
>
> `解释`：获取生成的Od曲线坐标集，传入的字段为曲线的分段数，默认值是`20`
>
>
> ### setOptions
>
> `描述`：setOptions({Object}options)
>
> `解释`：修改坐标数组等属性
>
> 看到`getPoints`方法，我直呼牛B，这开发者是知道使用者想要什么的。

接下来再来看看lushu是如何运动的。

首先直接看构造方法：

```js
  var LuShu = (BMapGLLib.LuShu = function (map, path, opts) {
    if (!path || path.length < 1) {
      return;
    }
    this._map = map;
    if (opts["geodesic"]) {
      this._path = getGeodesicPath(path);
    } else {
      this._path = path;
    }
    this.i = 0;
    this._setTimeoutQuene = [];
    this._opts = { icon: null, speed: 400, defaultContent: "" };
    if (!opts["landmarkPois"]) {
      opts["landmarkPois"] = [];
    }
    this._setOptions(opts);
    this._rotation = 0;
    if (!(this._opts.icon instanceof BMapGL.Icon)) {
      this._opts.icon = defaultIcon;
    }
  });
```

大致做了以下几件事：

1. 设置好path，也就是`this._path`，具体用处在后面。
2. 设置了一些字段，比如`this.i`
3. 设置了`opts`

再来看我们让lushu开始时调用的`lushu.start()`：

```js
  LuShu.prototype.start = function () {
    var me = this,
      len = me._path.length;
    if (me.i && me.i < len - 1) {
      if (!me._fromPause) {
        return;
      } else {
        if (!me._fromStop) {
          me._moveNext(++me.i);
        }
      }
    } else {
      me._addMarker();
      me._timeoutFlag = setTimeout(function () {
        me._addInfoWin();
        if (me._opts.defaultContent == "") {
          me.hideInfoWindow();
        }
        me._moveNext(me.i);
      }, 400);
    }
    this._fromPause = false;
    this._fromStop = false;
  };
```

判断了一下从什么状态开始start的，我们直接看最后一个else里的内容：

首先把图标（marker）添加进来，然后设置了一个setTimeout，具体内容是把infowindow添加进来，然后执行了`me._moveNext(me.i)`，这个函数就是所有的关键点了。我们先明确me.i是怎么来的，事实上它就是`this.i`，也就是在构造函数中设置的一个字段，我们进到`_moveNext`中来看其含义：

```js
    _moveNext: function (index) {
      var me = this;
      if (index < this._path.length - 1) {
        me._move(me._path[index], me._path[index + 1], me._tween.linear);
      }
    },
```

到这里就很明显了，在构造函数中构造的`path`存放的是各个点(事实上，它是一个[{lng, lat}]类型的数组)，而`i`则是用来标注当前已经走到第几个点。我们进到`_move`中去看到底是如何运动的。

```js
    _move: function (initPos, targetPos, effect) {
      var me = this,
        currentCount = 0,
        timer = 10,
        step = this._opts.speed / (1000 / timer),
        init_pos = BMapGL.Projection.convertLL2MC(initPos),
        target_pos = BMapGL.Projection.convertLL2MC(targetPos);
      init_pos = new BMapGL.Pixel(init_pos.lng, init_pos.lat);
      target_pos = new BMapGL.Pixel(target_pos.lng, target_pos.lat);
      var mcDis = me._getDistance(init_pos, target_pos);
      var direction = null;
      if (mcDis > 30037726) {
        if (target_pos.x < init_pos.x) {
          target_pos.x += WORLD_SIZE_MC;
          direction = "right";
        } else {
          target_pos.x -= WORLD_SIZE_MC;
          direction = "left";
        }
      }
      var count = Math.round(me._getDistance(init_pos, target_pos) / step);
      if (count < 1) {
        me._moveNext(++me.i);
        return;
      }
      me._intervalFlag = setInterval(function () {
        if (currentCount >= count) {
          clearInterval(me._intervalFlag);
          if (me.i > me._path.length) {
            return;
          }
          me._moveNext(++me.i);
        } else {
          currentCount++;
          var x = effect(init_pos.x, target_pos.x, currentCount, count),
            y = effect(init_pos.y, target_pos.y, currentCount, count),
            pos = BMapGL.Projection.convertMC2LL(new BMapGL.Point(x, y));
          if (pos.lng > 180) {
            pos.lng = pos.lng - 360;
          }
          if (pos.lng < -180) {
            pos.lng = pos.lng + 360;
          }
          if (currentCount == 1) {
            var proPos = null;
            if (me.i - 1 >= 0) {
              proPos = me._path[me.i - 1];
            }
            if (me._opts.enableRotation == true) {
              me.setRotation(proPos, initPos, targetPos, direction);
            }
            if (me._opts.autoView) {
              if (!me._map.getBounds().containsPoint(pos)) {
                me._map.setCenter(pos);
              }
            }
          }
          if (me._opts.autoCenter) {
            me._map.setCenter(pos, { noAnimation: true });
          }
          me._marker.setPosition(pos);
          me._setInfoWin(pos);
        }
      }, timer);
    },
```

`_move`的源码比较长，其实总共只分为两段：

1. 设置一些变量，比如:

   * currentCount
   * timer
   * step
   * count

   然后通过计算获得一些值，比如pos相关的部分。

2. 设置运动，也就是setInterval里面的部分。

设置的这些变量似乎有些让人摸不着头脑，我们来分析一下。首先突破口是`timer`这个量，因为它在`setInterval`中被直接用到了，含义比较明确：代表着**每次执行**的间隔时间。那么这段代码的终止条件是什么呢，我们来看：

```js
        if (currentCount >= count) {
          clearInterval(me._intervalFlag);
          if (me.i > me._path.length) {
            return;
          }
          me._moveNext(++me.i);
        } else {
            currentCount++;
            ...
        }
```

可以看到，事实上是**每次执行**都会使`currentCount`自增，直到等于`count`，我们再回过头来看`count`的定义：

```js
    step = this._opts.speed / (1000 / timer),  
	var count = Math.round(me._getDistance(init_pos, target_pos) / step);
```

我们来列式子算一下：
$$
\text{step} = \frac{\text{speed}}{1000}\times \text{timer}
$$
由于`timer`是常量，我们可以写成：
$$
\text{step} \propto \text{speed}
$$
那么
$$
\text{count}=\frac{\text{distance}}{\text{step}}
$$
这是什么，这可不就是
$$
t=\frac{s}{v}
$$
嘛，所以说可以简单理解为：

* `count`表示完成这段运动一共需要多少"帧"；
* `currentCount`表示现在运动到第几"帧"了；
* `timer`表示运动一帧所需要的时间(ms)；
* `step`只是一个中间量；

继续往下分析：

```js
        } else {
          currentCount++;
          var x = effect(init_pos.x, target_pos.x, currentCount, count),
            y = effect(init_pos.y, target_pos.y, currentCount, count),
            pos = BMapGL.Projection.convertMC2LL(new BMapGL.Point(x, y));
			...
          me._marker.setPosition(pos);
          me._setInfoWin(pos);
        }
      }, timer);
    },
```

中间省略的是与运动直接关系不太大的（关于rotation后面会讲）部分，可以看到其实和我们猜测的一样，每次执行都通过effect函数来获得下一帧的坐标，然后调用`setPosition()`来修改位置，这样就可以做出动效来了。

有了这些之后，我们简单的思路就是：既然Arc使用了OdCurve，我们只需要在lushu中也得到同样的点路径，然后通过修改effect方法来在每帧中获取对应的点坐标即可。

#### 1. 构造OdCurve

首先引入mapvgl:

```js
var mapvgl_1 = require("mapvgl");
```

在lushu的构造方法中完全仿照Arc构造一个点列出来即可：

```js
	...
    const MAX_FRAME = 300;   
	if (opts["geodesic"]) {
      this._path = getGeodesicPath(path);
    }
    else if (opts["odCurve"]){  // 是否使用Odcurve
      var lineData = [];
      var curve = new mapvgl_1.OdCurve();
      for(var i = 0 ; i < path.length-1 ; i++){
        var start = path[i];
        var end = path[i+1];
        curve.setOptions({
          points: [start, end]
        });
        var curveModelData = curve.getPoints(MAX_FRAME-1); //最细
        lineData.push(curveModelData)
      }
      this.lineData = lineData;
      this._path = path;
    }
    else {
      this._path = path;
    }
	...
```

>  这里有个小trick:我使用了`MAX_FRAME-1`作为`getPoints`的参数，来获取到长度为`MAX_FRAME`的点列，这么做是因为我想要让lushu在每段Arc的运动时长相等，而不是速度相等，避免非常短和非常长的Arc在同一个展示中，导致lushu非常鸡肋。
>
> 因为lushu是通过currentCount来控制的，如果要每段都定时的话其实非常简单，只需要固定下来count就可以了。这样的话如果我需要更改这个时长，也可以通过设置count来实现。
>
> 而`MAX_FRAME`其实是为了方便其它时长的情况方便地获取到点列，直接通过(0, MAX_FRAME)到(0, count)的映射就可以获得到对应的点，而不需要每针对一个count就重新获取一个点列。

#### 2. 修改effect

事实上，effect函数是一个回调函数，它在`_moveNext`中被传入：

```js
    _moveNext: function (index) {
      var me = this;
      if (index < this._path.length - 1) {
        me._move(me._path[index], me._path[index + 1], me._tween.linear);
          // me._tween.linear
      }
    },
```

我们找到`linear`：

```js
    _tween: {
      linear: function (initPos, targetPos, currentCount, count) {
        var b = initPos;
        var c = targetPos - initPos;
        var t = currentCount;
        var d = count;
        return (c * t) / d + b;
      }
    },
```

emmmm,也许当时开发的时候是想过拓展多种方式的，只是没实现留了个接口而已。正好我们也用得上，只是需要魔改一下：

```js
    _tween: {
      linear: function (initPos, targetPos, currentCount, count) {
        ...
      },
      OdCurve: function (currentCount, count, lineData, i) {
        var lineDataArrayIndex = Math.round(MAX_FRAME*currentCount/count)
        return lineData[i][lineDataArrayIndex>=MAX_FRAME?MAX_FRAME-1:lineDataArrayIndex]
      }
    },
```

`linear`我这里就直接弃用了，所以参数也重新写了，这些参数的含义都已经解释过了，函数体本身你也只做了一件非常简单的事：求出对应的映射点，然后把该点直接返回。

当然在effect的调用处我们也需要小改一下(_move的setInterval里)：

```js
        } else {
          currentCount++; // 下一帧
          // var x = effect(init_pos.x, target_pos.x, currentCount, me.speed, "x"),
          //   y = effect(init_pos.y, target_pos.y, currentCount, me.speed, "y"),
          var nextPoint = effect(currentCount, me.speed, me.lineData, i);
          var x = nextPoint[0],
            y = nextPoint[1],
            pos = window.BMapGL.Projection.convertMC2LL(new window.BMapGL.Point(x, y));
            ...
```

就完成了。此时会发现，确实按照轨迹运动了，但是旋转非常鬼畜，令人匪夷所思。我们再来看关于Rotation的部分：

```js
            if (me._opts.enableRotation == true) {
              me.setRotation(proPos, initPos, targetPos, direction);
            }
```

这里给`setRotation`传进去了四个参数，意义比较明确（proPos应该是prePos打错了，但是问题也不大，反正都是proPos不影响运行，而且实际上这个变量都没被用过，也不知道什么原因），我们直接来看`setRotation`:

```js
    setRotation: function (prePos, curPos, targetPos, direction) {
      var me = this;
      var deg = 0;
      curPos = me._map.pointToPixel(curPos);
      targetPos = me._map.pointToPixel(targetPos);
      if (targetPos.x != curPos.x) {
        var tan = (targetPos.y - curPos.y) / (targetPos.x - curPos.x),
          atan = Math.atan(tan);
        deg = (atan * 360) / (2 * Math.PI);
        if ((!direction && targetPos.x < curPos.x) || direction === "left") {
          deg = -deg + 90 + 90;
        } else {
          deg = -deg;
        }
        me._marker.setRotation(-deg);
      } else {
        var disy = targetPos.y - curPos.y;
        var bias = 0;
        if (disy > 0) {
          bias = -1;
        } else {
          bias = 1;
        }
        me._marker.setRotation(-bias * 90);
      }
      return;
    },
```

其实也很好理解，看到tan和atan大概就明白是直接把方向改成两个点的连线方向。但是为甚么我们使用会出问题呢，原因是传入的是这段线的起始点和终点两个点，而不是我们魔改过后的路程点列中的每个点，所以只需要在`_move`开始的时候设置一个新的字段`currentPos`：

```
    _move: function (initPos, targetPos, effect, i) {
      var me = this,
        currentCount = 0,
        currentPos = initPos,
        ...
```

然后再把传入的参数改成

```js
          if (me._opts.enableRotation == true) {
            me.setRotation(prePos, currentPos, pos, direction);
            currentPos = pos;
          }
```

就可以了。

*By JSYRD*