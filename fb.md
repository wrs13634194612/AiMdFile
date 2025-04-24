说明：我计划用angular，做一个即时通讯的功能，socket实现长连接，让客户端和服务端可以发送和接收消息，实时响应，并且增加了心跳检测和断线重连的机制

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f742e8d25366483e8f388b55e9e70db0.png#pic_center)

json格式：这是自己约定的json协议的格式，无论发送消息，还是心跳检测，都是用这个

```bash
{
    "version": "1.0.0",
    "event": "system",
    "payload": {
        "id": "sys_nD9Vl1jOt_qnjNN_AAAD",
        "timestamp": 1741061899758,
        "content": "连接成功",
        "meta": {
            "connection": true
        }
    },
    "code": 201
}
```

step1:首先，我们需要写一个js的服务，放在package.json同级别的目录下，然后在终端运行此js，相当于服务

C:\Users\Administrator\WebstormProjects\untitled4\server.js

```javascript
const express = require('express');
const { createServer } = require('http');
const { Server } = require('socket.io');

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, {
  cors: {
    origin: "http://localhost:4200",
    methods: ["GET", "POST"]
  }
});

// 生成标准消息格式
const createMessage = (content, sender = null) => ({
  version: '1.0.0',
  event: 'message',
  payload: {
    id: `msg_${Date.now()}_${Math.random().toString(36).substr(2, 5)}`,
    timestamp: Date.now(),
    sender: sender ? { id: sender.id, name: sender.name } : undefined,
    content,
    meta: { protocol: 'websocket' }
  },
  code: 200
});

io.on('connection', (socket) => {
  // 发送连接成功消息
  const systemMessage = {
    version: '1.0.0',
    event: 'system',
    payload: {
      id: `sys_${socket.id}`,
      timestamp: Date.now(),
      content: '连接成功',
      meta: { connection: true }
    },
    code: 201
  };
  socket.emit('message', systemMessage);

  // 新增心跳监听
  socket.on('heartbeat', (data) => {
    socket.emit('heartbeat_ack', { timestamp: data.timestamp });
  });

  // 处理客户端消息
  socket.on('message', (rawData) => {
    console.log('服务端收到客户端的消息',rawData)
    try {
      // 验证数据格式
      const data = typeof rawData === 'string' ? JSON.parse(rawData) : rawData;

      if (!data?.version || data.version !== '1.0.0') {
        throw new Error('协议版本不匹配');
      }
      // 构造响应消息
      const response = createMessage(`已收到：${data.payload.content}`, {
        id: 'server',
        name: '系统服务'
      });
      // 广播给所有客户端
      io.emit('message', response);

    } catch (error) {
      // 错误处理
      const errorMessage = {
        version: '1.0.0',
        event: 'error',
        payload: {
          id: `err_${Date.now()}`,
          timestamp: Date.now(),
          content: error.message,
          meta: { errorCode: 'INVALID_FORMAT' }
        },
        code: 400
      };
      socket.emit('message', errorMessage);
    }
  });
});

httpServer.listen(3000, () => {
  console.log('服务端运行在 http://localhost:3000');
});

```

step2:打开终端，执行这条命令

```bash
S C:\Users\Administrator\WebstormProjects\untitled4> node server.js
```
提示，服务端运行在 http://localhost:3000，表示服务启动成功

step3:接下来，我需要约定传输协议的json格式，包括消息和心跳
C:\Users\Administrator\WebstormProjects\untitled4\src\app\wechat\socket-types.ts

```javascript
export interface MessageProtocol {
  version: '1.0.0';
  event: 'message' | 'system' | 'error';
  payload: {
    id: string;
    timestamp: number;
    sender?: {
      id: string;
      name: string;
    };
    content: string;
    meta?: Record<string, any>;
  };
  code: number;
}

export type MessageEvent = 'message' | 'system' | 'heartbeat' | 'heartbeat_ack';

```

step4:接下来，我需要写一个socket服务，来管理发送消息，接收消息，心跳检测等功能
C:\Users\Administrator\WebstormProjects\untitled4\src\app\wechat\socket.service.ts

```javascript
import { Injectable, NgZone } from '@angular/core';
import { io, Socket } from 'socket.io-client';
import { Observable, Subject } from 'rxjs';
import { MessageProtocol } from './socket-types';

const HEARTBEAT_INTERVAL = 5000;
const HEARTBEAT_TIMEOUT = 10000;

@Injectable({ providedIn: 'root' })
export class SocketService {
  private socket!: Socket;
  private heartbeatTimer: any;
  private timeoutTimer: any;
  public connectionState = new Subject<boolean>();

  constructor(private ngZone: NgZone) {
    this.initializeSocket();
  }

  private initializeSocket() {
    this.socket = io('http://localhost:3000', {
      transports: ['websocket'],
      autoConnect: false
    });

    this.setupSocketListeners();
  }

  private setupSocketListeners() {
    this.socket.on('connect', () => this.handleConnect());
    this.socket.on('disconnect', (reason) => this.handleDisconnect(reason));
    this.socket.on('heartbeat_ack', () => this.clearTimeoutTimer());
  }

  /* 心跳检测和重连逻辑 */
  private startHeartbeat() {
    this.heartbeatTimer = setInterval(() => {
      this.socket.emit('heartbeat', { timestamp: Date.now() });
      this.startTimeoutTimer();
    }, HEARTBEAT_INTERVAL);
  }

  private startTimeoutTimer() {
    this.timeoutTimer = setTimeout(() => {
      console.warn('心跳超时，强制断开连接');
      this.socket.disconnect();
    }, HEARTBEAT_TIMEOUT);
  }

  /* 公共方法 */
  public connect() {
    this.socket.connect();
  }

  public disconnect() {
    this.socket.disconnect();
  }

  public sendMessage(content: string) {
    const message: MessageProtocol = {
      version: '1.0.0',
      event: 'message',
      payload: {
        id: `client_${Date.now()}`,
        timestamp: Date.now(),
        content,
        meta: { platform: 'web' }
      },
      code: 200
    };
    console.log('sendMessage', message)
    this.socket.emit('message', message);
  }

  public listen(event: 'message' | 'system'): Observable<MessageProtocol> {
    return new Observable(subscriber => {
      this.socket.on(event, (data: MessageProtocol) => {
        console.log('resultMessage', data)
        if (this.validateMessage(data)) subscriber.next(data);
      });
    });
  }

  /* 辅助方法 */
  private handleConnect() {
    this.ngZone.run(() => {
      this.connectionState.next(true);
      this.startHeartbeat();
    });
  }

  private handleDisconnect(reason: string) {
    this.ngZone.run(() => {
      console.log('连接断开:', reason);
      this.connectionState.next(false);
      this.clearTimers();
    });
  }

  private validateMessage(data: any): data is MessageProtocol {
    return !!data?.version && !!data?.payload?.content;
  }

  private clearTimers() {
    clearInterval(this.heartbeatTimer);
    this.clearTimeoutTimer();
  }

  private clearTimeoutTimer() {
    clearTimeout(this.timeoutTimer);
  }
}

```

step5:  C:\Users\Administrator\WebstormProjects\untitled4\src\app\wechat\wechat.component.ts

```javascript
import { Component , OnInit, OnDestroy} from '@angular/core';
import { SocketService } from './socket.service';
import { MessageProtocol } from './socket-types';
import { Subscription } from 'rxjs';
import {FormsModule} from '@angular/forms';
import {NgForOf} from '@angular/common';

import { CommonModule } from '@angular/common';


@Component({
  selector: 'app-wechat',
  imports: [
    FormsModule,
    CommonModule,
    NgForOf
  ],
  templateUrl: './wechat.component.html',
  styleUrls: ['./wechat.component.css']
})
export class WechatComponent implements OnInit, OnDestroy {
  connectionStatus = false;
  inputMessage = '';
  messages: MessageProtocol[] = [];

  private statusSub!: Subscription;
  private subs = new Subscription();

  constructor(private socketService: SocketService) {}

  ngOnInit() {
    this.initConnectionStatus();
    this.initMessageListeners();
    this.socketService.connect();
  }

  private initConnectionStatus() {
    this.statusSub = this.socketService.connectionState.subscribe(connected => {
      this.connectionStatus = connected;
    });
  }

  private initMessageListeners() {
    this.subs.add(
      this.socketService.listen('message').subscribe({
        next: msg => this.handleNewMessage(msg),
        error: err => console.error('消息接收错误', err)
      })
    );

    this.subs.add(
      this.socketService.listen('system').subscribe({
        next: sysMsg => this.handleSystemMessage(sysMsg)
      })
    );
  }

  sendMessage() {
    if (this.inputMessage.trim()) {
      try {
        this.socketService.sendMessage(this.inputMessage);
        this.inputMessage = '';
      } catch (error) {
        console.error('消息发送失败:', error);
        alert('消息发送失败，请检查网络连接');
      }
    }
  }

  private handleNewMessage(msg: MessageProtocol) {
    this.messages = [...this.messages, msg];
    this.autoScrollToBottom();
  }

  private autoScrollToBottom() {
    setTimeout(() => {
      const container = document.querySelector('.message-list');
      if (container) container.scrollTop = container.scrollHeight;
    }, 50);
  }

  private handleSystemMessage(sysMsg: MessageProtocol) {
    this.messages = [...this.messages, sysMsg];
    if (sysMsg.code >= 500) {
      alert(`系统错误: ${sysMsg.payload.content}`);
    }
  }

  ngOnDestroy() {
    this.subs.unsubscribe();
    this.statusSub.unsubscribe();
    this.socketService.disconnect();
  }
}

```

step6: C:\Users\Administrator\WebstormProjects\untitled4\src\app\wechat\wechat.component.html

```html
<div class="chat-container">
  <!-- 连接状态指示 -->
  <div class="connection-status" [class.connected]="connectionStatus">
    {{ connectionStatus ? '🟢 已连接' : '🔴 连接断开' }}
  </div>

  <!-- 消息输入区域 -->
  <div class="input-section">
    <input
      type="text"
      [(ngModel)]="inputMessage"
      placeholder="输入消息..."
      (keyup.enter)="sendMessage()"
      class="message-input"
    >
    <button
      (click)="sendMessage()"
      class="send-button"
      [disabled]="!inputMessage.trim()"
    >
      发送
    </button>
  </div>

  <!-- 消息展示区域 -->
  <div class="message-list">
    <div *ngFor="let msg of messages" class="message-item">
      <div class="message-header">
        <span class="timestamp">
          {{ msg.payload.timestamp | date:'yyyy-MM-dd HH:mm:ss' }}
        </span>
        <span *ngIf="msg.payload.sender" class="sender">
          {{ msg.payload.sender.name }}:
        </span>
      </div>
      <div class="message-content">
        {{ msg.payload.content }}
        <span *ngIf="msg.event === 'system'" class="system-tag">[系统]</span>
        <span *ngIf="msg.event === 'error'" class="error-tag">[错误]</span>
      </div>
    </div>
  </div>
</div>

```

step7:C:\Users\Administrator\WebstormProjects\untitled4\src\app\wechat\wechat.component.css

```css
.chat-container {
  max-width: 800px;
  margin: 20px auto;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.connection-status {
  padding: 8px 12px;
  border-radius: 4px;
  margin-bottom: 16px;
  background: #dc3545;
  color: white;
}

.connection-status.connected {
  background: #28a745;
}

.input-section {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.message-input {
  flex: 1;
  padding: 12px;
  border: 1px solid #ced4da;
  border-radius: 6px;
  font-size: 16px;
}

.message-input:focus {
  outline: none;
  border-color: #86b7fe;
  box-shadow: 0 0 0 3px rgba(13, 110, 253, 0.25);
}

.send-button {
  padding: 12px 24px;
  background: #007bff;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  transition: background 0.3s;
}

.send-button:hover {
  background: #0056b3;
}

.send-button:disabled {
  background: #6c757d;
  cursor: not-allowed;
}

.message-list {
  border: 1px solid #dee2e6;
  border-radius: 6px;
  background: white;
  max-height: 500px;
  overflow-y: auto;
}

.message-item {
  padding: 16px;
  border-bottom: 1px solid #eee;
}

.message-item:last-child {
  border-bottom: none;
}

.message-header {
  display: flex;
  gap: 10px;
  margin-bottom: 8px;
  font-size: 0.9em;
  color: #6c757d;
}

.message-content {
  font-size: 16px;
  line-height: 1.5;
  padding-left: 20px;
}

.system-tag {
  color: #28a745;
  margin-left: 10px;
  font-size: 0.8em;
}

.error-tag {
  color: #dc3545;
  margin-left: 10px;
  font-size: 0.8em;
}

```


end