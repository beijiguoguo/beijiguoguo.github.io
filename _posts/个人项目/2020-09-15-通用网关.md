# 通用网关

> 基于springboot2.0.5，undertow，jooq，mysql，redis，okhttp，jwt实现的网关，可代理http请求，websocket以及sse，实现动态四级限流（服务、接口、IP、用户），服务降级，负载均衡以及IP黑白名单。服务配置从mysql读取，定时刷新配置，可动态修改配置，定时器刷新即可生效。
>
> 服务不断完善中......



- [x] http api
- [x] websocket
- [x] sse
- [x] 身份认证
- [x] 限流
- [x] 降级
- [x] 负载均衡
- [x] IP黑白名单