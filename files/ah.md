说明：

 MQTT 异步通信系统功能文档
1. 系统概述
本系统基于 MQTT 协议实现异步通信，包含三个核心组件：

Broker（消息代理）：负责消息的路由和转发。
Client（主客户端）：定时发送时间戳消息并等待响应。
Echo Client（回显客户端）：接收消息并原样返回。
所有组件均运行在本地（localhost），使用端口 10008 进行通信。

2. 功能描述
2.1 Broker（消息代理）
持续运行，负责接收和转发消息。
监听 localhost:10008，确保客户端之间的通信畅通。
2.2 Client（主客户端）
每秒发布当前时间戳到 /ping 主题。
订阅 /pong 主题，等待回显消息。
检测连接状态，在断开时终止运行。
2.3 Echo Client（回显客户端）
订阅 /ping 主题，接收主客户端发送的消息。
将接收到的消息原样发布到 /pong 主题。
检测连接状态，在断开时终止运行。
3. 通信流程
Client 发送消息
每秒生成时间戳，发布到 /ping 主题。
Echo Client 接收并回显
从 /ping 获取消息，原样转发到 /pong 主题。
Client 接收回显
从 /pong 获取消息，打印并进入下一轮循环。
如果任一客户端断开连接，系统会检测并终止运行。

4. 运行机制
异步架构：使用 asyncio 实现非阻塞并发，提高效率。
自动重连：客户端连接失败时，以 100ms 间隔尝试重连。
日志输出：各组件打印关键操作（如发送/接收消息），便于调试。
5. 适用场景
MQTT 协议学习：演示基本的发布/订阅机制。
设备间通信模拟：测试消息收发逻辑。
异步编程实践：展示 asyncio 与 MQTT 的结合使用。
系统设计简洁，便于扩展或集成到更复杂的项目中。




包含功能：
MQTT服务器
运行在本地端口10008，支持匿名访问
数据存储于内存中，无持久化
程序终止时自动关闭服务
Ping客户端
每秒发送一条时间戳消息到主题/ping
订阅主题/pong，接收并显示该主题的消息
Echo客户端
监听主题/ping，收到消息后立即将内容转发至主题/pong
实现自动应答机制





/////我是分割线

python部分

step101:C:\Users\wangrusheng\PycharmProjects\FastAPIProject1\hello.py

```python

import asyncio
import time

import mqttools

BROKER_PORT = 10008


async def start_client():
    client = mqttools.Client('localhost', BROKER_PORT, connect_delays=[0.1])
    await client.start()

    return client


async def client_main():
    """Publish the current time to /ping and wait for the echo client to
    publish it back on /pong, with a one second interval.

    """

    client = await start_client()
    await client.subscribe('/pong')

    while True:
        print()
        message = str(int(time.time())).encode('ascii')
        print(f'client: Publishing {message} on /ping.')
        client.publish(mqttools.Message('/ping', message))
        message = await client.messages.get()
        print(f'client: Got {message.message} on {message.topic}.')

        if message is None:
            print('Client connection lost.')
            break

        await asyncio.sleep(1)


async def echo_client_main():
    """Wait for the client to publish to /ping, and publish /pong in
    response.

    """

    client = await start_client()
    await client.subscribe('/ping')

    while True:
        message = await client.messages.get()
        print(f'echo_client: Got {message.message} on {message.topic}.')

        if message is None:
            print('Echo client connection lost.')
            break

        print(f'echo_client: Publishing {message.message} on /pong.')
        client.publish(mqttools.Message('/pong', message.message))


async def broker_main():
    """The broker, serving both clients, forever.

    """

    broker = mqttools.Broker(('localhost', BROKER_PORT))
    await broker.serve_forever()


async def main():
    await asyncio.gather(
        broker_main(),
        echo_client_main(),
        client_main()
    )


asyncio.run(main())


```

step102:运行

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject1> python hello.py 

client: Publishing b'1744796556' on /ping.
echo_client: Got b'1744796556' on /ping.
echo_client: Publishing b'1744796556' on /pong.
client: Got b'1744796556' on /pong.

client: Publishing b'1744796557' on /ping.
echo_client: Got b'1744796557' on /ping.
echo_client: Publishing b'1744796557' on /pong.
client: Got b'1744796557' on /pong.

client: Publishing b'1744796558' on /ping.
echo_client: Got b'1744796558' on /ping.
echo_client: Publishing b'1744796558' on /pong.
client: Got b'1744796558' on /pong.

```

end

/////我是分割线

step201:C:\Users\wangrusheng\IdeaProjects\untitled2\build.gradle

```bash
plugins {
    id 'java'
}

group = 'org.example'
version = '1.0-SNAPSHOT'

repositories {
    mavenCentral()
}

dependencies {
    // MQTT 服务端依赖
    implementation 'io.moquette:moquette-broker:0.15'
    // MQTT 客户端依赖
    implementation 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.2.5'

    testImplementation platform('org.junit:junit-bom:5.10.0')
    testImplementation 'org.junit.jupiter:junit-jupiter'
}

test {
    useJUnitPlatform()
}
```

step202:C:\Users\wangrusheng\IdeaProjects\untitled2\src\main\java\org\example\Main.java

```java
package org.example;

import org.eclipse.paho.client.mqttv3.MqttException;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class Main {
    public static void main(String[] args) {
        // 启动MQTT服务器
        startMqttServer();

        // 等待服务器初始化
        try {
            Thread.sleep(1500);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        // 启动客户端
        startPingClient();
        startEchoClient();
    }

    private static void startMqttServer() {
        new Thread(() -> {
            try {
                System.out.println("Starting MQTT server...");
                MqttServer.startServer();
            } catch (Exception e) {
                System.err.println("Failed to start server: ");
                e.printStackTrace();
                System.exit(1);
            }
        }).start();
    }

    private static void startPingClient() {
        new Thread(() -> {
            try {
                MqttClientHandler client = new MqttClientHandler("ping-client");
                client.connect();
                client.subscribe("/pong", 0);

                // 定时发送ping消息
                ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();
                scheduler.scheduleAtFixedRate(() -> {
                    try {
                        String timestamp = String.valueOf(System.currentTimeMillis() / 1000);
                        System.out.printf("client: Publishing %s on /ping.\n", timestamp);
                        client.publish("/ping", timestamp, 0);
                    } catch (MqttException e) {
                        e.printStackTrace();
                    }
                }, 0, 1, TimeUnit.SECONDS);

                // 保持线程运行
                Thread.sleep(Long.MAX_VALUE);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
    }

    private static void startEchoClient() {
        new Thread(() -> {
            try {
                MqttClientHandler echoClient = new EchoClientHandler("echo-client");
                echoClient.connect();
                echoClient.subscribe("/ping", 0);

                // 保持线程运行
                Thread.sleep(Long.MAX_VALUE);
            } catch (Exception e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

step203:C:\Users\wangrusheng\IdeaProjects\untitled2\src\main\java\org\example\EchoClientHandler.java

```java
package org.example;

import org.eclipse.paho.client.mqttv3.MqttMessage;

public class EchoClientHandler extends MqttClientHandler {
    public EchoClientHandler(String clientId) {
        super(clientId);
    }

    @Override
    public void messageArrived(String topic, MqttMessage message)   {
        System.out.printf("echo_client: Got %s on %s.\n", new String(message.getPayload()), topic);

        try {
            if ("/ping".equals(topic)) {
                String payload = new String(message.getPayload());
                System.out.printf("echo_client: Publishing %s on /pong.\n", payload);

                publish("/pong", payload, 0);
            }
        }catch (Exception e){
            e.printStackTrace();
        }

    }
}
```

step204:C:\Users\wangrusheng\IdeaProjects\untitled2\src\main\java\org\example\MqttServer.java

```java
package org.example;

import io.moquette.broker.Server;
import io.moquette.broker.config.IConfig;
import io.moquette.broker.config.MemoryConfig;
import java.io.IOException;
import java.util.Properties;

public class MqttServer {
    private static Server mqttBroker;

    public static void startServer() throws IOException {
        if (mqttBroker == null) {
            mqttBroker = new Server();
            IConfig config = new MemoryConfig(serverConfig());
            mqttBroker.startServer(config);
            System.out.println("MQTT Broker started on port 10008");
            addShutdownHook();
        }
    }

    private static Properties serverConfig() {
        Properties props = new Properties();
        props.put("port", "10008"); // 修改端口
        props.put("host", "0.0.0.0");
        props.put("allow_anonymous", "true");
        props.put("persistence_store", "memory");
        return props;
    }


    private static void addShutdownHook() {
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.out.println("Stopping MQTT broker...");
            mqttBroker.stopServer();
            System.out.println("MQTT broker stopped");
        }));
    }
}
```

step205:C:\Users\wangrusheng\IdeaProjects\untitled2\src\main\java\org\example\MqttClientHandler.java

```java
package org.example;

import org.eclipse.paho.client.mqttv3.*;
import org.eclipse.paho.client.mqttv3.persist.MemoryPersistence;
import java.util.concurrent.Executors;
import java.util.concurrent.ScheduledExecutorService;
import java.util.concurrent.TimeUnit;

public class MqttClientHandler implements MqttCallbackExtended {
    private static final String BROKER_URL = "tcp://localhost:10008";
    private final String clientId;
    private IMqttClient client;
    private final ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

    public MqttClientHandler(String clientId) {
        this.clientId = clientId;
    }

    public void connect() throws MqttException {
        client = new MqttClient(BROKER_URL, clientId, new MemoryPersistence());
        client.setCallback(this);
        MqttConnectOptions options = new MqttConnectOptions();
        options.setAutomaticReconnect(true);
        options.setCleanSession(true);
        options.setConnectionTimeout(30);
        client.connect(options);
    }

    // 其他方法保持相同，只需修改messageArrived的日志输出
    @Override
    public void messageArrived(String topic, MqttMessage message) {
        System.out.printf("client: Got %s on %s.\n", new String(message.getPayload()), topic);
    }

    // 其他原有方法保持不变...




    public void publish(String topic, String content, int qos) throws MqttException {
        MqttMessage message = new MqttMessage(content.getBytes());
        message.setQos(qos);
        client.publish(topic, message);
    }

    public void subscribe(String topic, int qos) throws MqttException {
        client.subscribe(topic, qos);
    }

    public void disconnect() throws MqttException {
        client.disconnect();
        client.close();
        scheduler.shutdown();
    }

    @Override
    public void connectionLost(Throwable cause) {
        System.out.println("Connection lost, attempting reconnect...");
        scheduler.scheduleAtFixedRate(() -> {
            try {
                if (!client.isConnected()) {
                    client.reconnect();
                    System.out.println("Reconnected successfully");
                }
            } catch (MqttException e) {
                System.err.println("Reconnect failed: " + e.getMessage());
            }
        }, 0, 5, TimeUnit.SECONDS);
    }

    @Override
    public void connectComplete(boolean reconnect, String serverURI) {
        System.out.println("Connection established: " + (reconnect ? "Reconnected" : "New connection"));
    }


    @Override
    public void deliveryComplete(IMqttDeliveryToken token) {
        System.out.println("Message delivery confirmed");
    }


}
```

step206：运行

```bash
ing ':org.example.Main.main()'…

> Task :compileJava UP-TO-DATE
> Task :processResources NO-SOURCE
> Task :classes UP-TO-DATE

> Task :org.example.Main.main()
Starting MQTT server...
MQTT Broker started on port 10008
log4j:WARN No appenders could be found for logger (io.moquette.broker.Server).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Connection established: New connection
Connection established: New connection
client: Publishing 1744797021 on /ping.
Message delivery confirmed
echo_client: Got 1744797021 on /ping.
echo_client: Publishing 1744797021 on /pong.
client: Publishing 1744797022 on /ping.
Message delivery confirmed
client: Got 1744797021 on /pong.
client: Publishing 1744797023 on /ping.
Message delivery confirmed
client: Publishing 1744797024 on /ping.
Message delivery confirmed
client: Publishing 1744797025 on /ping.
Message delivery confirmed
client: Publishing 1744797026 o
```

end