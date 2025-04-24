说明：用fastapi+tcp+android实现在线聊天，测试完成
效果图：
客户端1：

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject> python client.py
请输入用户名: a

输入消息（格式：@用户ID 内容 或 直接输入内容）: 
==================================================
在线用户列表：
  0c868691 | a
==================================================

[2025-03-15 08:26:31] 系统通知: a 进入聊天室

==================================================
在线用户列表：
  0c868691 | a
  1add816b | 
==================================================

[2025-03-15 08:26:36] 系统通知:  进入聊天室

==================================================
在线用户列表：
  0c868691 | a
==================================================

[2025-03-15 08:26:41] 系统通知:  已离开

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
==================================================

[2025-03-15 08:26:47] 系统通知: b 进入聊天室

[2025-03-15 08:26:52] 公共消息 b: hello
good boy

输入消息（格式：@用户ID 内容 或 直接输入内容）:
[2025-03-15 08:27:05] 公共消息 a: good boy

[2025-03-15 08:27:25] 私聊来自 b: what is your name
@466947dd i name is bob

输入消息（格式：@用户ID 内容 或 直接输入内容）:
[2025-03-15 08:27:46] 私聊来自 a: i name is bob

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
  74cfb061 | UserA
==================================================

[2025-03-15 08:40:37] 系统通知: UserA 进入聊天室

[2025-03-15 08:40:39] 公共消息 UserA: Hello everyone, this is UserA

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
==================================================

[2025-03-15 08:40:43] 系统通知: UserA 已离开

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
  432066ef | UserA
==================================================

[2025-03-15 08:41:20] 系统通知: UserA 进入聊天室

[2025-03-15 08:41:21] 公共消息 UserA: Hello everyone, this is UserA

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
==================================================

[2025-03-15 08:41:25] 系统通知: UserA 已离开


```

客户端2：

```bash
(.venv) PS C:\Users\wangrusheng\PycharmProjects\FastAPIProject> python client.py
请输入用户名: b

输入消息（格式：@用户ID 内容 或 直接输入内容）: 
==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
==================================================

[2025-03-15 08:26:47] 系统通知: b 进入聊天室
hello

输入消息（格式：@用户ID 内容 或 直接输入内容）: 
[2025-03-15 08:26:52] 公共消息 b: hello

[2025-03-15 08:27:05] 公共消息 a: good boy
@0c868691 what is your name

输入消息（格式：@用户ID 内容 或 直接输入内容）: 
[2025-03-15 08:27:25] 私聊来自 b: what is your name

[2025-03-15 08:27:46] 私聊来自 a: i name is bob

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
  74cfb061 | UserA
==================================================

[2025-03-15 08:40:37] 系统通知: UserA 进入聊天室

[2025-03-15 08:40:39] 公共消息 UserA: Hello everyone, this is UserA

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
==================================================

[2025-03-15 08:40:43] 系统通知: UserA 已离开

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
  432066ef | UserA
==================================================

[2025-03-15 08:41:20] 系统通知: UserA 进入聊天室

[2025-03-15 08:41:21] 公共消息 UserA: Hello everyone, this is UserA

==================================================
在线用户列表：
  0c868691 | a
  466947dd | b
==================================================

[2025-03-15 08:41:25] 系统通知: UserA 已离开


```
step1:C:\Users\wangrusheng\PycharmProjects\FastAPIProject\main.py

```python
import asyncio
import json
import uuid
import datetime
import logging
from typing import Dict

# 配置日志
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger("ChatServer")

class ConnectionManager:
    def __init__(self):
        self.active_connections: Dict[str, dict] = {}

    async def connect(self, reader: asyncio.StreamReader, writer: asyncio.StreamWriter, client_id: str, username: str):
        self.active_connections[client_id] = {
            "reader": reader,
            "writer": writer,
            "username": username,
            "addr": writer.get_extra_info('peername')
        }
        logger.info(f"用户 {username}({client_id}) 已连接")
        await self._broadcast_user_list()
        await self._send_system_message(f"{username} 进入聊天室")

    async def disconnect(self, client_id: str):
        if client_id in self.active_connections:
            user = self.active_connections[client_id]
            user["writer"].close()
            del self.active_connections[client_id]
            logger.info(f"用户 {user['username']}({client_id}) 已断开")
            await self._broadcast_user_list()
            await self._send_system_message(f"{user['username']} 已离开")

    async def handle_message(self, sender_id: str, data: dict):
        if data["type"] == "private":
            await self._send_private_message(
                sender_id=sender_id,
                recipient_id=data["to"],
                content=data["content"]
            )
        else:
            await self._broadcast_message(sender_id, data["content"])

    async def _send_private_message(self, sender_id: str, recipient_id: str, content: str):
        sender = self.active_connections.get(sender_id)
        recipient = self.active_connections.get(recipient_id)

        if sender and recipient:
            message = {
                "type": "private",
                "from": sender_id,
                "to": recipient_id,
                "sender_name": sender["username"],
                "content": content,
                "timestamp": self._current_time()
            }
            await self._send_to_client(recip_id=recipient_id, message=message)
            await self._send_to_client(recip_id=sender_id, message=message)  # 回显

    async def _broadcast_message(self, sender_id: str, content: str):
        sender = self.active_connections.get(sender_id)
        if sender:
            message = {
                "type": "public",
                "from": sender_id,
                "sender_name": sender["username"],
                "content": content,
                "timestamp": self._current_time()
            }
            for client_id in self.active_connections:
                await self._send_to_client(client_id, message)

    async def _send_system_message(self, content: str):
        message = {
            "type": "system",
            "content": content,
            "timestamp": self._current_time()
        }
        for client_id in self.active_connections:
            await self._send_to_client(client_id, message)

    async def _broadcast_user_list(self):
        users = [{
            "client_id": cid,
            "username": info["username"]
        } for cid, info in self.active_connections.items()]

        message = {
            "type": "user_list",
            "users": users
        }
        for client_id in self.active_connections:
            await self._send_to_client(client_id, message)

    async def _send_to_client(self, recip_id: str, message: dict):
        try:
            writer = self.active_connections[recip_id]["writer"]
            data = json.dumps(message) + "\n"
            writer.write(data.encode())
            await writer.drain()
        except (KeyError, ConnectionError):
            await self.disconnect(recip_id)

    def _current_time(self):
        return datetime.datetime.now().strftime("%Y-%m-%d %H:%M:%S")

manager = ConnectionManager()

async def handle_client(reader: asyncio.StreamReader, writer: asyncio.StreamWriter):
    client_id = str(uuid.uuid4())[:8]
    try:
        # 接收初始化信息
        data = await reader.readuntil(b"\n")
        init_data = json.loads(data.decode().strip())
        username = init_data.get("username", f"用户{client_id}")

        await manager.connect(reader, writer, client_id, username)

        while True:
            data = await reader.readuntil(b"\n")
            msg = json.loads(data.decode().strip())
            await manager.handle_message(client_id, msg)

    except (asyncio.IncompleteReadError, json.JSONDecodeError):
        logger.error("收到无效数据")
    except ConnectionResetError:
        logger.info("客户端强制断开连接")
    finally:
        await manager.disconnect(client_id)
        writer.close()

async def main():
    server = await asyncio.start_server(
        handle_client, 
        host="0.0.0.0",
        port=8000
    )
    async with server:
        await server.serve_forever()

if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        logger.info("服务器已关闭")
```

step2:C:\Users\wangrusheng\PycharmProjects\FastAPIProject\client.py

```python
import asyncio
import json
import sys


class ChatClient:
    def __init__(self):
        self.reader = None
        self.writer = None
        self.client_id = ""
        self.username = ""

    async def connect(self, host: str, port: int):
        self.reader, self.writer = await asyncio.open_connection(host, port)
        self.username = input("请输入用户名: ")
        await self._send_init_message()

    async def _send_init_message(self):
        init_msg = {"username": self.username}
        await self._send_message(init_msg)

    async def _send_message(self, msg: dict):
        data = json.dumps(msg) + "\n"
        self.writer.write(data.encode())
        await self.writer.drain()

    async def receive_messages(self):
        try:
            while True:
                data = await self.reader.readuntil(b"\n")
                msg = json.loads(data.decode().strip())
                self.handle_message(msg)
        except (asyncio.IncompleteReadError, ConnectionResetError):
            print("\n连接已断开")
            sys.exit(1)

    def handle_message(self, msg: dict):
        msg_type = msg["type"]
        timestamp = msg.get("timestamp", "")

        if msg_type == "user_list":
            print("\n" + "=" * 50)
            print("在线用户列表：")
            for user in msg["users"]:
                print(f"  {user['client_id']} | {user['username']}")
            print("=" * 50)

        elif msg_type == "private":
            sender = msg["sender_name"]
            content = msg["content"]
            print(f"\n[{timestamp}] 私聊来自 {sender}: {content}")

        elif msg_type == "public":
            sender = msg["sender_name"]
            content = msg["content"]
            print(f"\n[{timestamp}] 公共消息 {sender}: {content}")

        elif msg_type == "system":
            print(f"\n[{timestamp}] 系统通知: {msg['content']}")

    async def input_handler(self):
        while True:
            msg = await asyncio.get_event_loop().run_in_executor(
                None,
                input,
                "\n输入消息（格式：@用户ID 内容 或 直接输入内容）: "
            )

            if msg.lower() == 'exit':
                self.writer.close()
                return

            if msg.startswith("@"):
                parts = msg.split(" ", 1)
                if len(parts) == 2:
                    user_id, content = parts[0][1:], parts[1]
                    await self._send_message({
                        "type": "private",
                        "to": user_id,
                        "content": content
                    })
                    continue

            await self._send_message({
                "type": "public",
                "content": msg
            })


async def main():
    client = ChatClient()
    await client.connect("192.168.1.2", 8000)

    tasks = [
        asyncio.create_task(client.receive_messages()),
        asyncio.create_task(client.input_handler())
    ]

    await asyncio.gather(*tasks)


if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\n客户端已退出")
```

step3:C:\Users\wangrusheng\AndroidStudioProjects\MyApplication9\app\src\test\java\com\example\myapplication\MyFirstTest.kt

```java
package com.example.myapplication
import kotlinx.coroutines.*
import java.io.BufferedReader
import java.io.InputStreamReader
import java.io.PrintWriter
import java.net.Socket
import java.util.*
import com.google.gson.Gson

fun main() = runBlocking {
    val client = TcpChatTester("UserA", "127.0.0.1", 8000)
    launch { client.start() }

    // 测试操作序列
    delay(1500) // 等待连接建立
    client.sendPublicMessage("Hello everyone, this is UserA")
    delay(1000)
    client.sendPrivateMessage("UserB", "Hi B, this is a private message")
    delay(3000)

    client.close()
}



class TcpChatTester(
    private val username: String,
    private val host: String,
    private val port: Int
) {
    private var socket: Socket? = null
    private var writer: PrintWriter? = null
    private var reader: BufferedReader? = null
    private val gson = Gson()
    private val scope = CoroutineScope(Dispatchers.IO)
    private var isRunning = true

    fun start() {
        scope.launch {
            try {
                // 建立TCP连接
                socket = Socket(host, port)
                writer = PrintWriter(socket!!.getOutputStream(), true)
                reader = BufferedReader(InputStreamReader(socket!!.getInputStream()))

                // 发送初始化消息
                sendInitMessage()

                // 启动消息接收协程
                launch(Dispatchers.IO) {
                    receiveMessages()
                }

                println("[$username] Connection established")
            } catch (e: Exception) {
                println("[$username] Connection error: ${e.message}")
            }
        }
    }

    private fun sendInitMessage() {
        val initMsg = mapOf("username" to username)
        sendRawMessage(initMsg)
    }

    private fun sendRawMessage(message: Map<String, Any>) {
        writer?.println(gson.toJson(message))
    }

    fun sendPublicMessage(content: String) {
        val message = mapOf(
            "type" to "public",
            "content" to content
        )
        sendRawMessage(message)
        println("[$username] Sent public message: $content")
    }

    fun sendPrivateMessage(targetUser: String, content: String) {
        val message = mapOf(
            "type" to "private",
            "to" to targetUser,
            "content" to content
        )
        sendRawMessage(message)
        println("[$username] Sent private to $targetUser: $content")
    }

    private fun receiveMessages() {
        try {
            while (isRunning) {
                val json = reader?.readLine() ?: break
                handleMessage(json)
            }
        } catch (e: Exception) {
            println("[$username] Connection closed: ${e.message}")
        } finally {
            close()
        }
    }

    private fun handleMessage(json: String) {
        val data = gson.fromJson(json, Map::class.java)
        when (data["type"]) {
            "user_list" -> {
                val users = data["users"] as List<Map<String, String>>
                println("\n[$username] Online users updated:")
                users.forEach { println("  ${it["client_id"]} - ${it["username"]}") }
            }
            "private" -> {
                println("\n[$username] Received private from ${data["sender_name"]}: ${data["content"]}")
            }
            "public" -> {
                println("\n[$username] Received public message from ${data["sender_name"]}: ${data["content"]}")
            }
            "system" -> {
                println("\n[$username] System notification: ${data["content"]}")
            }
        }
    }

    fun close() {
        isRunning = false
        runBlocking {
            delay(100)
            writer?.close()
            reader?.close()
            socket?.close()
            scope.cancel()
            println("[$username] Connection closed")
        }
    }
}
```

end