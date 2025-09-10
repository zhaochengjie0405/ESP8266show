# ESP8266show


sequenceDiagram
    participant User
    participant Phone as 用户手机
    participant ESP as ESP8266 (AP模式)
    participant STM as STM32
    participant Cloud as 阿里云

    Note over ESP, STM: 设备启动，等待配网
    STM->>ESP: AT+CWSAP (启动AP热点)
    ESP-->>Phone: 热点可见 "MyDevice_Config"

    User->>Phone: 连接设备热点
    User->>Phone: 浏览器访问 192.168.4.1
    Phone->>ESP: HTTP GET / 请求
    ESP->>STM: 转发HTTP请求数据
    STM->>ESP: 发送包含表单的HTTP响应
    ESP->>Phone: 返回配置页面(含WiFi/MQTT等参数表单)

    User->>Phone: 填写表单并提交
    Phone->>ESP: HTTP POST 请求 (包含所有参数数据)
    ESP->>STM: 转发POST数据: "ssid=...&mqtt_host=..."

    Note over STM: 关键解析步骤
    STM->>STM: 解析查询字符串
    STM->>STM: 提取并存储所有参数(WiFi/MQTT/自定义)
    STM->>ESP: AT+CWJAP (连接用户路由器)
    STM->>ESP: AT+MQTTCONN (连接阿里云)

    STM->>Cloud: 连接成功，开始通信
    Cloud-->>STM: 订阅成功/发布确认
    STM->>Phone: 返回配置成功页面
