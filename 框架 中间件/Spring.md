### ApplicationContextAware 应用
> didi内部用，代替直接注入@Bean，而是直接获取对应的class

return SpringContextUtils.getBean(PreSchedulePayManager.class);
[Spring- ApplicationContextAware实际应用 - 掘金 (juejin.cn)](https://juejin.cn/post/6930502287689252877)

过滤器 & 拦截器区别：https://www.bilibili.com/video/BV1Zu411Y7W2