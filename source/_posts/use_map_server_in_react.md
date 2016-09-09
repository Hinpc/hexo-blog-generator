title: 在 React 中使用地图服务  
date: 2016-09-05 21:01  
modified: 2016-09-07 20:40  
tag:
 - React
 - 地图

photos:
 - /img/2016/20160905-ukraine-map-1422625.jpg
---

项目中用到了地图服务，以高德地图 JavaScript API 为例记录了部分代码，其他地图服务使用方法类似。需求：异步加载地图，增删点标记。

<!--more-->

本文默认你已学习了地图服务的 JavaScript API 使用方法。

## 公共模块:

```js
// utils.js
export const loadMapResource = () => new Promise((resolve, reject) => {
  const key = 'replaceyourkeyhere';
  const script = document.createElement('script');

  document.body.appendChild(script);

  script.onload = () => resolve(script);
  script.onerror = () => reject(new Error('load map error'));

  script.src = `http://webapi.amap.com/maps?v=1.3&key=${key}`;
});

export const getAmap = (domId, initPoint) => new AMap.Map(domId, {
  zoom: 13,
  center: [initPoint.longitude, initPoint.latitude],
});
```
`loadMapResource` 利用 `Promise` 来异步加载地图， `getAmap` 用来在 `domId` 这个节点上创建一个地图。
注意要在 `onload` 事件之前将 script 插入到 body 中去。参考： http://stackoverflow.com/questions/16230886/trying-to-fire-onload-event-on-script-tag

## 调用部分：

```js
// myMap.js
import React from 'react';
import { loadMapResource, getAmap } from 'utils';

class Map extends React.Component {
  componentDidMount() {
    const domId = 'mapContainer';
    const initPoint = {
      longitude: 121.382379,
      latitude: 31.232876,
    };

    // 按需加载地图
    if (!window.AMap) {
      loadMapResource().then(() => {
        this.map = getAmap(domId, initPoint);
      });
    } else {
      this.map = getAmap(domId, initPoint);
    }
  }

  componentDidUpdate() {
    // 如果需要， 可以添加 if 判断
    this.updateMarker();
  }

  updateMarker() {
    const addMarkerToMap = (position) => {
      ...
      const marker = new AMap.Marker({
        ...
        position: [position.longitude, position.latitude],
      });

      marker.setMap(this.map);
      this.markerStore.push(marker);
      ...
    };

    // reset
    this.map.remove(this.markerStore);
    // responseMarkers是从服务器请求得到的点标记集合，可以通过redux管理
    responseMarkers.forEach((item, index) => {
        const position = {
          longitude: item.longitude,
          latitude: item.latitude,
        };
        // addMarkerToMap用来将position标注到map上
        addMarkerToMap(position);
      });
    }
  }

  render() {
    return (
      <div className="map">
        <div id="mapContainer">
          地图加载中...
        </div>
        ...
      </div>
    );
  }
}
```

页面 mount 之后检查是否存在全局变量 `amap`，不存在则加载地图，加载地图后通过 `getAmap` 方法新建一个地图对象。

页面每次 update 时删除所有点标记，然后重新添加，添加过的点存储到 markerStore 集合（代码省掉了 constructor ）。因为我的项目中多个页面用到这些点，所以可以存储到 redux 中。

[本文完]
