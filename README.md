#### wepy 1.0V 框架 templave

```
npm install  安装依赖

npm run dev  开发环境
npm run test 测试环境
npm run build 打包生产环境
```

#### components文件夹
组件：wepy语法类似Vue


#### 这里的环境是如何配置的？
wepy.config.js

    ['global-define', {
      __NODE_ENV__: process.env.NODE_ENV,
      __VERSION__: '1.0.0',
      __AUTHOR__: 'Kip Song'
    }]
引入全局__NODE_ENV__的环境

##### config文件夹
1. config.dev.js
  开发环境配置
2. config.prod.js
  生产环境配置
3. config.test.js
  测试环境配置




#### server文件夹
请求接口API
建议根据多个模块/系统进行拆分

#### app.wpy
系统主入口等于Vue中main.js


#### request.js
##### 这里我们还是根据小程序的请求做了一层包装
##### 找到成功返回resolve，我们根据业务做了一层判断
##### 也做了login vilidate，如果不需要有个RETURN参数作为不校验
```javascript

export default {
  /**
   * @author Kip song
   * @param {*} params
   * @param {*} method
   */
  baseOptions(params, method = "GET") {
    return new Promise(function (resolve, reject) {
      let { url, data } = params
      let contentType = "application/json"
      contentType = params.contentType || contentType
      const option = {
        url: url.indexOf("http") !== -1 ? url : config.url + url,
        data: data,
        method: method,
        header: { 'content-type': contentType, 'access-token': getToken() },
        success(res) {
          CHECK_RESPONSE_ASYNC(res, option.url, data)
          if (res.statusCode === HTTP_STATUS.NOT_FOUND) {
            wx.hideLoading()
            wepy.showToast({
              title: '服务器找不到您所请求的文件或脚本！',
              icon: 'none',
              duration: 1500
            })
          } else if (res.statusCode === HTTP_STATUS.BAD_GATEWAY) {
            wx.hideLoading()
            wepy.showToast({
              title: '服务端异常，请稍后尝试！',
              icon: 'none',
              duration: 1500
            })
          } else if (res.statusCode === HTTP_STATUS.FORBIDDEN) {
            wx.hideLoading()
            wepy.showToast({
              title: '没有权限访问！',
              icon: 'none',
              duration: 1500
            })
          } else if (res.statusCode === HTTP_STATUS.SUCCESS) {
            let _data = res.data
            if (_data.state == 1 && _data.data) {
              resolve(_data.data)
            } else if (_data.state == 2) {
              if (data && data.RETURN) {
                console.warn('过滤拦截，不做处理')
              }
              else if (_data.code == 100018) {
                wx.navigateTo({
                  url: "../pages/login"
                })
              } else {
                wepy.showToast({
                  title: _data.msg || _data.message || `错误状态${_data.code}`,
                  icon: 'none',
                  duration: 1500
                })
              }
            } else if (_data.state == 3) {
              wepy.showToast({
                title: "未登录，请重新登录！",
                icon: 'none',
                duration: 1500
              })
            }
          } else {
            wx.hideLoading()
            wepy.showToast({
              title: '网络出现故障，请重新尝试！',
              icon: 'none',
              duration: 1500
            })
          }
        },
        fail(e) {
          CHECK_NETWORK_OUT()
          wepy.showToast({
            title: '网络超时，请重新操作！',
            icon: 'none',
            duration: 1500
          })
        }
      }
      wx.request(option)
    })
  },
  /**
     * GET请求
     * @param url 请求路径
     * @param data 参数
     */
  get(url, data) {
    let option = { url, data }
    return this.baseOptions(option, "GET")
  },
  /**
   * POST请求
   * @param url 请求路径
   * @param data 参数
   */
  post(url, data) {
    let option = { url, data }
    return this.baseOptions(option, "POST")
  },
  /**
   * PUT请求
   * @param url 请求路径
   * @param data 参数
   */
  put(url, data) {
    let option = { url, data }
    return this.baseOptions(option, "PUT")
  },
  /**
   * DELETE请求
   * @param url 请求路径
   * @param data 参数
   */
  delete(url, data) {
    let option = { url, data }
    return this.baseOptions(option, "DELETE")
  }
}
```

