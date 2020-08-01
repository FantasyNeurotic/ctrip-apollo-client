

随着业务使用的持续迭代，可以 [watch](https://github.com/lvgithub/stick#readme) 持续关注哦, 点击 [star](https://github.com/lvgithub/stick#readme) 是对我们最大的支持与鼓励。

## Introduction

<img src="https://tva1.sinaimg.cn/large/007S8ZIlly1ghbnklwk6dj30va0k8teo.jpg" alt="image-20200718092438648" style="zoom:50%;" />

中心配置 `mysql.host:127.0.0.1`  客户端会自动转化为Json `{ mysql:{ host: 127.0.0.1 } } }` 方便处理。

## apollo 服务端测试环境:

* host: http://106.54.227.205/
* 账号: `apollo`
* 密码: `admin`

## Features
* 配置热更新
* 缓存配置到本地
* 灰度发布
* 支持 TypeScript

## Install
```
npm i @lvgithub/ctrip-apollo-client --save
```
## Usage

* 本demo 已经在测试环境创建项目  apolloclient，大家可以直接在本地测试；
* 在配置中心修改user.name值后，无需重启，再次请求，会自动取到最新的值；
* 如想体验建议下载demo运行[源码](./example/ts-demo/index.ts)（TypeScript）;
* JS版本demo [源码](./example/js-demo/index.js)

```typescript
import { CtripApplloClient, value, hotValue } from 'ctrip-apollo-client';
import Koa from 'koa';

const apollo = new CtripApplloClient({
    configServerUrl: 'http://106.54.227.205:8080',
    appId: 'apolloclient',
    accessKey: '<your Apollo AccessKey>',
    configPath: './config/apolloConfig.json',
    namespaceList: ['application', 'development.qa']
});
const app = new Koa();

const run = async () => {
    // 初始化配置
    await apollo.init();

    // 获取的配置，不会热更新
    const port = apollo.getValue('app.port:3000');
    // 获取配置，支持热更新，需要通过 appName.value 获取最终值
    const appName = hotValue('app.name:apollo-demo');

    class User {
        // 通过装饰器注入，支持热更新
        // 只能注入类的属性
        @value("user.name:liuwei")
        public name: string
    }
    const user = new User();

    app.use(async (ctx, next) => {
        ctx.body = {
            appName: appName.value,
            userName: user.name
        }
        await next();
    })
    app.listen(port);
    console.log('listening on port:', port);
    console.info(`curl --location --request GET \'http://localhost:${port}\' `);
}
run();
```
[javascript demo 请点击链接](./example/js-demo/index.js)

## API

**new ApolloClient(options)** 构造函数
* returns: `apolloClient`

* options
    * **configServerUrl** `string` `required` Apollo配置服务的地址
    * **appId** `string` `required` 应用的appId
    * **accessKey** `string` Apollo AccessKey
    * **clusterName** `string` 集群名,默认值:`default`
    * **namespaceList** `array` Namespace的名字,默认值:`[application]`
    * **configPath** `string` 本地配置文件路径 默认值`./config/apolloConfig.json`
    * **logger** `object` 日志类 必须实现 `logger.info()`,`logger.error()` 两个方法

**init(timeoutMs)** 
初始化配置中心，拉取远端配置到端上，并且缓存一份到文件中，同时开启`HTTP Long Polling`来实时监控配置，如果拉取超过 timeoutMs ，或者发生异常，则读取本地缓存的配置文件。如果本地没有缓存配置文件，抛出异常。
* return: Promise<void>   
* timeoutMs 超时时间

**getConfigs()**  获取最新的配置文件

* returns: `object`
```
const config = apollo.getConfigs();
```

**getValue (namespace = 'application')**  获取具体的配置字段
* returns: `string`
* namespace 默认值: `application`
* field `string` eg: `mysql.port:3306` 分号前面key,如果未配置 3306 作为默认值
```
class User {
    get userName () {
        return apollo.getValue({ field: 'user.name:liuwei' });
    }
}
```
**hotValue (namespace = 'application')**  获取具体的配置字段，封装 `getter`(热更新)
* returns: `{value}` 
* namespace 默认值: `application`
* field `string` 属性位置 eg: `mysql.port:3306` 分号前面key,如果未配置 3306 作为默认值
```
const userName = apollo.hotValue({ field: 'user.name:liuwei' });
console.log(userName.value);
```

**withValue(target, key, field, namespace)**

* returns: `void` 
* target 目标对象
* key 需要注入对象的属性
* field `string` 属性位置 eg: `mysql.port:3306` 分号前面key,如果未配置 3306 作为默认值
* namespace `string` 默认值:`application`
  
```
class User {
    constructor () {
        withValue(this, 'userId', { field: 'user.id:10071' });
    }
}
// userId 属性会跟随配置更新
new User().userId
```

**onChange (callback(object))**  配置变更回调通知
* returns: `void`

**value(field, namespace)** 注入器，只能注入类的属性

* field `string` 字段属性
* namespace `string`
```
import { value } from 'ctrip-apollo-client';
class User {
    @value("user.name:liuwei")
    public name: string
}
```
## Benchmark
**一次性注入** [localValue] x **736,896,802** ops/sec ±1.49% (82 runs sampled)

**支持热更新** [hotValue] x **2,021,310** ops/sec ±1.28% (87 runs sampled)

**热更新默认值** [hotValue default] x **1,581,645** ops/sec ±0.89% (87 runs sampled)

**装饰器注入**  [decorator] x **2,161,312** ops/sec ±0.96% (87 runs sampled)

**原生访问** [dot] x **704,644,395** ops/sec ±1.45% (82 runs sampled)

Fastest is [localValue]


## License

[MIT](LICENSE)


