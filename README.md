# local-server-mock

* 前端开发，一般都是前端先根据产品prd或者设计稿求产出静态页面，然后再和后端联调，把页面变成动态的。

* 这个过程主要麻烦点就是当后端接口没有开发完成的时候，前端需要自己去模拟页面数据，后端接口开发完成的时候前端需要和后端联调真实的接口数据。

* mock主要解决数据模拟、接口反向代理问题

* 开发一般使用webpack，前端server用node启动的，因此mock被包装成了一个node中间件

## 如何使用？

```bash
npm i local-server-mock -D
```
<strong>PS：</strong> 中间件内置了mockjs，支持mockjs写法

## 使用方式

```js
    // 基础使用： 
    const mock = require('local-server-mock)();

    // 数据mock
    app.use(mock.reWriteResponseMiddleware)
```

```js
    // 自定义mock规则
    const mock = require('local-server-mock)({
        // mock默认位置
        mockFilePath: '/mock',
        // 匹配mock文件名的规则
        mockFileNameRules: /mock(-.+)?\.js$/,
        // browser类型路由配置，过滤前端路由
        exclude: [/^\/static/],
        // 多级请求路由转发，mock目录下配置文件
        proxyTable: 'proxyTable.js'
    });

    // 数据mock
    app.use(mock.reWriteResponseMiddleware)
```

## create-react-app使用

- 在scripts/start.js 添加代码

```js
//导入这个包
var mock = require('local-server-mock')()

//添加中间件
devServer.app.use(mock.reWriteResponseMiddleware);
```

## webpack中使用

```js
devServer: {
    after: app => {
        const Mock = require('local-server-mock')();
        app.use(Mock.reWriteResponseMiddleware);
    }
}
```

```js
//导入这个包
var mock = require('local-server-mock')()

//添加中间件
devServer.app.use(mock.reWriteResponseMiddleware);
```

## 配置Demo

```js
普通规则mock文件配置：
module.exports = {
    rules: [
        {
            pattern: /^\/api\/map\/trace\/configuration/,
            respondwith: 'map/configuration.json',
            sleep: 1000
        },
        {
            pattern: /^\/api\/map\/traces/,
            respondwith: 'map/traces.json',
            sleep: 1000
        }
    ]
};
```

```js
proxyTable文件配置：

module.exports = {
    rules: [
        {
            pattern: /\/pms\//,
            proxy: 'http://localhost:3001'
        }
    ]
};
```

## SfeMock

### API

| 参数名 | 说明 | 类型 | 默认值 | 必需 |
| -------- | ----------- | ---- | ------- | --------- |
| mockFilePath | mock文件夹位置 | string | '/mock/' | 否 |
| mockFileNameRules | mock文件规则 | RegExp | /mock(-.+)?\.js$/ | 否 |
| exclude | 需要排除的路径【一般在前端路由类型为browser类型需要标志出请求不拦截路由】 | Array | [] | 否 |
| proxyTable | proxyTable文件名 | string | '' | 否 |

### 单条规则配置

| 参数名 | 说明 | 类型 | 默认值 | 必需 |
| -------- | ----------- | ---- | ------- | --------- |
| pattern | 请求正则 | RegExp/String | * | 是 |
| respondwith | 响应mock | String | mock响应，规则 | 是 |
| sleep | 响应延迟，单位为毫秒 | number | 500 | 否 |

### proxyTable规则配置

| 参数名 | 说明 | 类型 | 默认值 | 必需 |
| -------- | ----------- | ---- | ------- | --------- |
| pattern | 请求正则 | RegExp/String | * | 是 |
| proxy | 代理机器 | String | '' | 是 |

```js
/**
 *  当请求/user/getinfo时，后端会返回user/user.json里面的内容
 * 
 *  当请求/user/login时，后端返回当前js进行处理后的结果
 **/
module.exports = {
    "rules": [{
        "pattern": /^\\/user\\/getinfo/,
        "respondwith": "user/user.json"
    },
    {
        "pattern": /^\\/user\\/login/,
        "respondwith": "user/user.js"
    }
};
```

```js
/**
 *  本地mock的json、js文件类型，内置了mockjs，支持mockjs语法
 * 
 **/
{
    "errno: 0", 
    "data|1": [1,2,3,4,5,6]
}
```

## 关于代理配置与优先级

- 1、文件中配置respondwith绝对路径形式：比如http://didi.com/

- 2、文件中单规则配置proxy参数： proxy: http://didi.com

- 3、命令行参数形式：proxyTable配置文件中转发规则 + node start --proxyTable 

- 4、命令行参数形式：node start --proxy http://didi.com

* 优先级规则： 1 > 2 > 3 > 4