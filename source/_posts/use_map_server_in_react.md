title: 在 React 中使用地图服务  
date: 2016-09-05 21:01  
modified: 2016-11-15 20:40  
tag:
- React
- 地图

photos:
- /img/2016/20160905-ukraine-map-1422625.jpg

---

项目中用到了地图服务，以高德地图 JavaScript API 为例记录了部分代码，其他地图服务使用方法类似。需求：异步加载地图，增删点标记。

<!--more-->

> 2016.11.15 修改异步加载地图 API 的方法

## 地图组件:
```jsx
// amap.jsx
import React from 'react';
import lazyLoadMapApi from './lib/lazyLoadMapApi';
import guid from './lib/guid';

class AMap extends React.Component {
  componentDidMount() {
    const mapElements = document.querySelectorAll('.el-react-amap-container .el-react-amap');
    const loadConfig = {
      key: 'yourkey',
    };
    let elementId;

    lazyLoadMapApi(loadConfig).then(() => {
      Array.from(mapElements).forEach(($el, index) => {
        elementId = guid(); // 生成随机的id
        $el.id = elementId;
        this[`mapInstance${index}`] = new window.AMap.Map(elementId);
      })
    });
  }

  render() {
    return (
      <div className="el-react-amap-container" style={styles.container}>
        <div className="el-react-amap" style={styles.map}></div>
      </div>
    );
  }
}

const styles = {
  container: {
    width: '100%',
    height: '100%',
  },
  map: {
    width: '100%',
    height: '100%',
  },
};

export default AMap;
```

在地图的 api 加载完成后，找到 `.el-react-amap` 节点，并赋值一个随机的 id 属性，以供生成地图实例时使用。



## 异步加载API

```javascript
// lazyLoadMapApi.js
const DEFAULT_CONFIG = {
  key: null,
  v: 1.3,
  protocol: window.location.protocol || 'https:',
  hostAndPath: 'webapi.amap.com/maps',
  plugin: [],
  callback: 'mapInitCallback',
};

const lazyLoadMapApi = (config = DEFAULT_CONFIG) => {
  const _config = { ...DEFAULT_CONFIG, ...config };
  const getScriptSrc = (cfg) => {
    let scriptSrc = `${cfg.protocol}//${cfg.hostAndPath}?v=${cfg.v}&key=${cfg.key}&callback=${cfg.callback}`;
    if (cfg.plugin.length) scriptSrc += `&plugin=${cfg.plugin.join(',')}`;
    return scriptSrc;
  };

  if (window.AMap) return Promise.resolve();

  const script = document.createElement('script');
  script.type = 'text/javascript';
  script.async = true;
  script.defer = true;
  script.src = getScriptSrc(_config);

  const scriptLoadingPromise = new Promise((resolve, reject) => {
    // api 加载成功的回调函数
    window[_config.callback] = () => {
      return resolve();
    };
    script.onerror = error => reject(error);
  });
  document.head.appendChild(script);
  return scriptLoadingPromise;
};

export default lazyLoadMapApi;
```

JavaScript API 入口脚本中加入 `callback` 参数，脚本加载完毕则会执行 window 下的 `callback` 指向函数。

---

参考：

http://stackoverflow.com/questions/16230886/trying-to-fire-onload-event-on-script-tag

https://github.com/ElemeFE/vue-amap

[异步加载地图](http://lbs.amap.com/api/javascript-api/example/map/asynchronous-loading-map/)
