# 哪吒面板

服务期状态监控，被动接收，极省资源 128M 小鸡也能装 Agent（非 node-exporter 那种主动拉取的方式。）

|  哪吒面板    |   首页截图1   |   首页截图2   |
| ---- | ---- | ---- |
|   <img src="https://s3.ax1x.com/2020/12/08/DzHv6A.jpg" width="2333px" />   | ![首页截图1](https://s3.ax1x.com/2020/12/07/DvTCwD.jpg)     | <img src="https://s3.ax1x.com/2020/12/09/rPF4xJ.png" width="1600px" /> |

\>> [查看针友列表](https://www.google.com/search?q=%22powered+by+%E5%93%AA%E5%90%92%E9%9D%A2%E6%9D%BF%22) (Google)

## 一键脚本

- 海外：

    ```shell
    curl -L https://raw.githubusercontent.com/naiba/nezha/master/script/install.sh -o nezha.sh && chmod +x nezha.sh
    sudo ./nezha.sh
    ```

- 国内加速：

    ```shell
    curl -L https://raw.sevencdn.com/naiba/nezha/master/script/install.sh -o nezha.sh && chmod +x nezha.sh
    sudo ./nezha.sh
    ```

## 使用说明
### 自定义 CSS

- 默认主题更改进度条颜色示例

    ```
    .ui.fine.progress> .bar {
        background-color: pink !important;
    }
    ```

- hotaru 主题更改背景图片示例

    ```
    .hotaru-cover {
        background: url(https://s3.ax1x.com/2020/12/08/DzHv6A.jpg) center;
    }
    ```

### 报警通知

#### 通知到 server酱 示例

1. 添加通知方式

    - 备注：server酱
    
    - URL：https://sc.ftqq.com/SCUrandomkeys.send
    
    - 请求方式: GET
    
    - 请求类型: JSON/FORM 都可以，其他接入其他API时要选择其使用的类型
    
    - Body: `{"text": "#NEZHA#"}`
      Body 参数必须是`JSON`，格式是 `key:value` 的形式，`#NEZHA#` 是面板消息占位符，面板触发通知时会自动替换占位符到实际消息
      
      请求方式为 GET 时面板会将 `Body` 里面的参数拼接到 URL 的 query 里面
    
2. 添加一个离线报警

    - 备注：离线通知
    - 规则：`[{"Type":"offline","Min":0,"Max":0,"Duration":10}]`
    - 启用：√

3. 添加一个监控 CPU 持续 10s 超过 50% **且** 内存持续 20s 占用低于 20% 的报警

    - 备注：CPU+内存
    - 规则：`[{"Type":"cpu","Min":0,"Max":50,"Duration":10},{"Type":"memory","Min":20,"Max":0,"Duration":20}]`
    - 启用：√

#### 报警规则说明

- Type
  - cpu、memory、swap、disk：Min/Max 数值为占用百分比
  - net_in_speed(入站网速)、net_out_speed(出站网速)、net_all_speed(双向网速)、transfer_in(入站流量)、transfer_out(出站流量)、transfer_all(双向流量)：Min/Max 数值为字节（1kb=1024，1mb = 1024*1024）
  - offline：不支持 Min/Max 参数
- Duration：持续秒数，监控比较简陋，取持续时间内的 70 采样结果

## 常见问题

### 数据备份恢复

数据储存在 `/opt/nezha` 文件夹中，迁移数据时打包这个文件夹，到新环境解压。然后执行一键脚本安装即可

### 启用 HTTPS

使用宝塔反代或者上CDN，建议 Agent配置 跟 访问管理面板 使用不同的域名，这样管理面板使用的域名可以直接套CDN，Agent配置的域名是解析管理面板IP使用的，也方便后面管理面板迁移（如果你使用IP，后面IP更换了，需要修改每个agent，就麻烦了）

### 反代配置

使用反向代理时需要针对 `/ws` 路径的 WebSocket 进行特别配置以支持实时更新服务器状态。

- Nginx(宝塔)：在你的 nginx 配置文件中加入以下代码

    ```nginx
    server{

        #server_name blablabla...

        location /ws {
            proxy_pass http://ip:站点访问端口;
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "Upgrade";
            proxy_set_header Host $host;
        }

        #其他的 location blablabla...
    }
    ```

- CaddyServer v1（v2无需特别配置）

    ```Caddyfile
    proxy /ws http://ip:8008 {
        websocket
    }
    ```

## 社区文章

- [哪吒探针 - Windows 客户端安装](https://nyko.me/2020/12/13/nezha-windows-client.html)
- [哪吒面板：小鸡们的最佳探针](https://www.zhujizixun.com/2843.html) *（已过时）*
