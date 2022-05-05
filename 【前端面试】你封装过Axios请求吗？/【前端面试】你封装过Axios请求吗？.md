# 前言

axios 是目前最优秀的 HTTP 请求库之一，虽然 axios 已经封装的非常好了，我们可以直接拿过来用。但是在实际的项目中，我们可能还需要对 axios 在封装一下，以便我们更好的管理项目和各个借口。

但是，目前网上有特别多的针对于 axios 在项目中的封装。不得不说，很多大佬封装得非常全面，方方面面都考虑到了。但是我们的每个真的都需要那些封装吗？显然不是的，网上的很多封装其实都显得**有点过度封装了**！

本篇文章实现最简单 Axios 封装，让小伙伴们扩展起来容易一些。

# 1.封装目的

此次进行简单的封装，所以暂时没有考虑**取消重复请求、重复发送请求、请求缓存**等情况！这里主要实现以下目的：

1. 实现请求拦截
2. 实现响应拦截
3. 常见错误信息处理
4. 请求头设置
5. api 集中式管理
# 2.初始化 axios 实例

虽然 axios 可以调用 get、post 等方法发起请求，但是我们为了更好的全局控制所有请求的相关配置，所以我们使用 axios.create()创建实例的方法来进行相关配置，这也是封装 axios 的精髓所在。

**示例代码：**

```javascript
// 创建 axios 请求实例
const serviceAxios = axios.create({
  baseURL: "", // 基础请求地址
  timeout: 10000, // 请求超时设置
  withCredentials: false, // 跨域请求是否需要携带 cookie
});
```
通过 create 方法我们得到了一个 axios 的实例，该实例上有很多方法，比如拦截器等等。我们创建实例的时候可以配置一些基础设置，比如基础请求地址，请求超时等等。
# 3.设置请求拦截

我们在发送请求的时候可能需要携带一些信息在请求头上，比如 token 等，所以说我们就需要将请求拦截下来，处理一些我们的业务逻辑。

**示例代码：**

```javascript
// 创建请求拦截
serviceAxios.interceptors.request.use(
  (config) => {
    // 如果开启 token 认证
    if (serverConfig.useTokenAuthorization) {
      config.headers["Authorization"] = localStorage.getItem("token"); // 请求头携带 token
    }
    // 设置请求头
    if(!config.headers["content-type"]) { // 如果没有设置请求头
      if(config.method === 'post') {
        config.headers["content-type"] = "application/x-www-form-urlencoded"; // post 请求
        config.data = qs.stringify(config.data); // 序列化,比如表单数据
      } else {
        config.headers["content-type"] = "application/json"; // 默认类型
      }
    }
    console.log("请求配置", config);
    return config;
  },
  (error) => {
    Promise.reject(error);
  }
);
```
我们通过调用 axios 的实例方法来进行请求拦截。
# 4.设置响应拦截

axios 请求的返回结果里面包含了很多东西，我们的业务层面通常只需要后端返回的数据即可，所以我们需要设置相应拦截，在响应结果返回给业务层之前做一些操作。

**示例代码：**

```javascript
// 创建响应拦截
serviceAxios.interceptors.response.use(
  (res) => {
    let data = res.data;
    // 处理自己的业务逻辑，比如判断 token 是否过期等等
    // 代码块
    return data;
  },
  (error) => {
    let message = "";
    if (error && error.response) {
      switch (error.response.status) {
        case 302:
          message = "接口重定向了！";
          break;
        case 400:
          message = "参数不正确！";
          break;
        case 401:
          message = "您未登录，或者登录已经超时，请先登录！";
          break;
        case 403:
          message = "您没有权限操作！";
          break;
        case 404:
          message = `请求地址出错: ${error.response.config.url}`;
          break;
        case 408:
          message = "请求超时！";
          break;
        case 409:
          message = "系统已存在相同数据！";
          break;
        case 500:
          message = "服务器内部错误！";
          break;
        case 501:
          message = "服务未实现！";
          break;
        case 502:
          message = "网关错误！";
          break;
        case 503:
          message = "服务不可用！";
          break;
        case 504:
          message = "服务暂时无法访问，请稍后再试！";
          break;
        case 505:
          message = "HTTP 版本不受支持！";
          break;
        default:
          message = "异常问题，请联系管理员！";
          break;
      }
    }
    return Promise.reject(message);
  }
);
```
# 5.完整示例

上面直接简单介绍了拦截等相关代码，接下来我们在实际项目中来演练一下，以 Vue 项目为例。

在 src 下面新建 http 文件夹，用来存储关于 axios 请求的一些文件，然后在 http 文件夹下新建 index.js 文件，用于封装我们的 axios，然后在新建 config 文件夹，主要用来创建配置文件，最后新建一个 api 文件夹，用于集中管理我们的接口。

**文件夹目录如下：**

![图片](data:image/png;base64,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)


**index.js 代码：**

```javascript
import axios from "axios";
import serverConfig from "./config";
import qs from "qs";

// 创建 axios 请求实例
const serviceAxios = axios.create({
  baseURL: serverConfig.baseURL, // 基础请求地址
  timeout: 10000, // 请求超时设置
  withCredentials: false, // 跨域请求是否需要携带 cookie
});

// 创建请求拦截
serviceAxios.interceptors.request.use(
  (config) => {
    // 如果开启 token 认证
    if (serverConfig.useTokenAuthorization) {
      config.headers["Authorization"] = localStorage.getItem("token"); // 请求头携带 token
    }
    // 设置请求头
    if(!config.headers["content-type"]) { // 如果没有设置请求头
      if(config.method === 'post') {
        config.headers["content-type"] = "application/x-www-form-urlencoded"; // post 请求
        config.data = qs.stringify(config.data); // 序列化,比如表单数据
      } else {
        config.headers["content-type"] = "application/json"; // 默认类型
      }
    }
    console.log("请求配置", config);
    return config;
  },
  (error) => {
    Promise.reject(error);
  }
);

// 创建响应拦截
serviceAxios.interceptors.response.use(
  (res) => {
    let data = res.data;
    // 处理自己的业务逻辑，比如判断 token 是否过期等等
    // 代码块
    return data;
  },
  (error) => {
    let message = "";
    if (error && error.response) {
      switch (error.response.status) {
        case 302:
          message = "接口重定向了！";
          break;
        case 400:
          message = "参数不正确！";
          break;
        case 401:
          message = "您未登录，或者登录已经超时，请先登录！";
          break;
        case 403:
          message = "您没有权限操作！";
          break;
        case 404:
          message = `请求地址出错: ${error.response.config.url}`;
          break;
        case 408:
          message = "请求超时！";
          break;
        case 409:
          message = "系统已存在相同数据！";
          break;
        case 500:
          message = "服务器内部错误！";
          break;
        case 501:
          message = "服务未实现！";
          break;
        case 502:
          message = "网关错误！";
          break;
        case 503:
          message = "服务不可用！";
          break;
        case 504:
          message = "服务暂时无法访问，请稍后再试！";
          break;
        case 505:
          message = "HTTP 版本不受支持！";
          break;
        default:
          message = "异常问题，请联系管理员！";
          break;
      }
    }
    return Promise.reject(message);
  }
);
export default serviceAxios;
```
**config/index.js 代码：**
```javascript
const serverConfig = {
  baseURL: "https://smallpig.site", // 请求基础地址,可根据环境自定义
  useTokenAuthorization: true, // 是否开启 token 认证
};
export default serverConfig;
```
**api/user.js 接口调用示例代码：**
```javascript
import serviceAxios from "../index";

export const getUserInfo = (params) => {
  return serviceAxios({
    url: "/api/website/queryMenuWebsite",
    method: "post",
    params,
  });
};
export const login = (data) => {
  return serviceAxios({
    url: "/api/user/login",
    method: "post",
    data,
  });
};
```
**注意：get 请求需要传 params，post 请求需要传 data。**
**Vue 文件中调用示例：**

```javascript
import { login } from "@/http/api/user"
async loginAsync() {
  let params = {
    email: "123",
    password: "12321"
  }
  let data = await login(params);
  console.log(data);
}
```
# 6.总结

我们这里只做了最最最基础的 axios 封装，但是可扩展性较高。相较于其它文章的过度封装，这里的封装形式其实可以满足大部分应用场景了。

我们在此基础上可以根据业务场景做一些以下扩展建议：

* 请求拦截里面针对 token 进行处理
* 响应拦截里面判断 token 是否过期等等
* 在 config/index.js 里面动态更改 baseURL
* 在请求拦截里面根据业务场景修改请求头
* 在拦截里面设置全局请求进度条等等
