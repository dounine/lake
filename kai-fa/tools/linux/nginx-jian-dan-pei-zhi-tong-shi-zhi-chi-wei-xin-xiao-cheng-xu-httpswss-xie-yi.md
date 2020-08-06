---
description: >-
  微信小程序需要使用https与wss能才进行连接，虽然开发模式下可以使用http与ws，但发布的时候还是需要安全协议,网上的各种配置是超多超复杂的，这里已经对nginx指定版本进行最简单的配置，可用。
---

# Nginx 简单配置同时支持微信小程序https/wss协议

nginx版本

```text
nginx -v
nginx version: nginx/1.12.2
```

配置

{% code title="/etc/nginx/conf.d/test.conf" %}
```text
server {
    listen   80;
    server_name test.dounine.com;
    return     301 https://$host$request_uri;
}

server {
    listen 443;
    server_name test.dounine.com;
    ssl on;
    ssl_certificate /etc/nginx/ssls/test.xxxx.pem;
    ssl_certificate_key /etc/nginx/ssls/test.xxxx.key;

    location / {
	    client_max_body_size    100m;
       	proxy_pass http://localhost:7777;
       	proxy_set_header  Host  $host;
       	proxy_set_header  X-Real-IP  $remote_addr;
       	proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
	    proxy_set_header Upgrade $http_upgrade;
	    proxy_set_header Connection "Upgrade";
    }

}
```
{% endcode %}

微信小程序代码

```javascript
wx.connectSocket({
  url: 'wss://test.dounine.com/ws'
});
wx.onSocketOpen(function(res) {
  console.info('websocket连接成功');
});
wx.onSocketClose(function(res) {
  console.log('WebSocket 已关闭！')
});
wx.onSocketError(function(res){
  console.log('WebSocket连接打开失败，请检查！')
});
wx.onSocketMessage(function(res) {
  console.log('收到服务器内容：' + res.data)
})
```

