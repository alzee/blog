Title:  title
Author: Al Zee  
Email:  z@alz.ee  
Web:    https://alz.ee  
Date:   Thu, 20 Jun 2024 13:41:19 +0800
Link:   https://alz.ee/article/miniprog_map.md
Tags:   

# 小程序手绘地图

客户的小程序用于景区展示。为了视觉效果，特意制作了精美的手绘地图，希望贴在标准地图上。效果类似于这样。


小程序中添加图层目前有3种方案：
1. 腾讯位置服务提供的[`个性化图层`](https://lbs.qq.com/customMap/)
1. 制作H5页面，调用各个地图服务的`自定义栅格图层`或`静态图贴地图层`接口，小程序中通过webview嵌套网页
1. 小程序接口[`MapContext.addGroundOverlay`](https://developers.weixin.qq.com/miniprogram/dev/api/media/map/MapContext.addGroundOverlay.html)

从技术角度讲，方案1是最优的，因为切成了瓦片地图，不同缩放级别显示不同尺寸的图片，效果也应该是最好的。但该功能要在小程序中使用，需要支付5万元/年[^1][^3][^2]。这对于大部分场景来说是不可接受的。   
方案2可以解决问题，但抛开性能不说，webview到H5，再通过[`JS-SDK`](https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html)到微信，兜了一圈，也实在算不上优雅。  

所以小程序端的方案就是`MapContext.addGroundOverlay`。

该接口主要参数`bounds`:
```ts
bounds: {
  southwest: {
    longitude: number,
    latitude: number
  },
  northeast: {
    longitude: number,
    latitude: number
  }
}
```
通过东北和西南两角的坐标，即可确定贴图覆盖面积，从而实现跟随地图一同缩放的效果。

[^1]: 现在是否支持地图瓦片（图层）工具配置至小程序地图上？: https://lbs.qq.com/FAQ/custom_faq.html
[^3]: 微信小程序上能否使用: https://lbs.qq.com/dev/console/customLayer/guide
[^2]: 如何申请商业授权？费用是多少呢？: https://lbs.qq.com/FAQ/authorization_faq.html
[^js-sdk]: js-sdk: https://developers.weixin.qq.com/doc/offiaccount/OA_Web_Apps/JS-SDK.html
