说明：我听说前端有一款数据库，叫IndexedDb数据库，可以存储超大的文件和数据，大约有250M，有了这个，就可以在浏览器里面，存储超大的数据，
事实上IndexedDb存储的数据，存在浏览器文件夹的c盘，缓存浏览器的文件夹里面，只要你存了数据，用户关闭浏览器，再次打开，数据依然还在，可以加载，用来做缓存，非常方便
因此，我就计划用angular框架，使用indexdb技术，先写一个简单的增删改查的小demo，用来存储用户信息，弄了一上午，总算可以用了，下面是详细步骤和效果图

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2a7b63d01bd74c48b044761031925588.png#pic_center)

step0:用npm安装IndexedDb

```bash
npm install ngx-indexed-db 
```

step1：IndexedDb安装成功后，在  package.json可以查看安装的IndexedDb版本19.3.3

```bash
 "@angular/animations": "^19.1.0",
    "@angular/cdk": "^19.1.5",
    "@angular/common": "^19.1.0",
    "@angular/compiler": "^19.1.0",
    "@angular/core": "^19.1.0",
    "@angular/forms": "^19.1.0",
    "@angular/material": "^19.1.5",
    "@angular/platform-browser": "^19.1.0",
    "@angular/platform-browser-dynamic": "^19.1.0",
    "@angular/router": "^19.1.0",
    "ngx-indexed-db": "^19.3.3",
    "rxjs": "~7.8.0",
    "tslib": "^2.3.0",
    "zone.js": "~0.15.0"
```

step2：接下来需要在angular项目中，导入IndexedDb
新建一个文件，文件名和具体路径是：C:\Users\Administrator\WebstormProjects\untitled4\src\app\db.config.ts

```javascript
import { DBConfig } from 'ngx-indexed-db';

export const DB_CONFIG: DBConfig = {
  name: 'Angular19DB',
  version: 2,
  objectStoresMeta: [{
    store: 'tasks',
    storeConfig: {
      keyPath: 'id',
      autoIncrement: true
    },
    storeSchema: [
      { name: 'title', keypath: 'title', options: { unique: false } },
      { name: 'status', keypath: 'status', options: { unique: false } }
    ]
  }]
};

```

step3:然后就是注册，
文件名和具体路径是：C:\Users\Administrator\WebstormProjects\untitled4\src\app\app.config.ts

```javascript
import { ApplicationConfig, provideZoneChangeDetection } from '@angular/core';
import { provideRouter } from '@angular/router';

import { routes } from './app.routes';
import { provideIndexedDb } from 'ngx-indexed-db';
import { DB_CONFIG } from './db.config';

import { provideAnimations } from '@angular/platform-browser/animations';
import { provideAnimationsAsync } from '@angular/platform-browser/animations/async';



export const appConfig: ApplicationConfig = {
  providers: [provideIndexedDb(DB_CONFIG),provideZoneChangeDetection({ eventCoalescing: true }), provideRouter(routes),provideAnimations(), provideAnimationsAsync()]
};

```

step4:路由这些，不用我多说了吧，新建一个模块，添加到路由里面，然后就直接开整，

```bash
 ng generate component student
```

step5:C:\Users\Administrator\WebstormProjects\untitled4\src\app\student\student.component.ts

```javascript
import { Component, OnInit, ChangeDetectorRef } from '@angular/core';
import { DatabaseService } from './data.service';
import { FormsModule } from '@angular/forms';
import { NgForOf } from '@angular/common';
import { Task } from './task.model';// 从统一定义文件导入

@Component({
  selector: 'app-task',
  imports: [FormsModule, NgForOf],
  templateUrl: './student.component.html',
  styleUrl: './student.component.css'
})

export class StudentComponent  implements OnInit {
  tasks: Task[] = [];
  newTask: Omit<Task, 'id'> = {
    title: '',
    status: 'pending'
  };

  constructor(
    private db: DatabaseService,
    private cdr: ChangeDetectorRef
  ) {}

  async ngOnInit() {
    await this.loadTasks();
  }

  async loadTasks() {
    // 添加类型断言确保数据一致性
    this.tasks = await this.db.getAllTasks() as Task[];
    this.cdr.detectChanges();
  }

  async addTask() {
    if (this.newTask.title.trim()) {
      // 类型安全传递
      await this.db.addTask(this.newTask);
      this.newTask = { title: '', status: 'pending' };
      await this.loadTasks();
    }
  }

  async updateTask(task: Task) {
    // 添加空值检查
    if (task.id === undefined) return;

    await this.db.updateTask(task);
    await this.loadTasks();
  }

  async deleteTask(id: number) {
    await this.db.deleteTask(id);
    await this.loadTasks();
  }
}

```

step6:C:\Users\Administrator\WebstormProjects\untitled4\src\app\student\task.model.ts

```javascript
// src/app/models/task.model.ts
export interface Task {
  id?: number;  // 将id改为可选属性
  title: string;
  status: 'pending' | 'completed';
}

```

step7:C:\Users\Administrator\WebstormProjects\untitled4\src\app\student\data.service.ts

```javascript
import { Injectable } from '@angular/core';
import { Task } from './task.model';

@Injectable({ providedIn: 'root' })
export class DatabaseService {
  private db!: IDBDatabase;
  private readonly dbName = 'AngularDB';
  private readonly storeName = 'tasks';

  // 添加数据库准备状态追踪
  private dbReady: Promise<void>;

  constructor() {
    this.dbReady = this.initializeDB();
  }

  private async initializeDB(): Promise<void> {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.dbName, 1);

      request.onupgradeneeded = (event) => {
        this.db = (event.target as IDBOpenDBRequest).result;
        if (!this.db.objectStoreNames.contains(this.storeName)) {
          const store = this.db.createObjectStore(this.storeName, {
            keyPath: 'id',
            autoIncrement: true
          });
          store.createIndex('title', 'title', { unique: false });
        }
      };

      request.onsuccess = (event) => {
        this.db = (event.target as IDBOpenDBRequest).result;
        resolve();
      };

      request.onerror = (event) => {
        reject((event.target as IDBRequest).error);
      };
    });
  }

  // 所有方法添加等待初始化
  async getAllTasks(): Promise<Task[]> {
    await this.dbReady;
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(this.storeName, 'readonly');
      const store = transaction.objectStore(this.storeName);
      const request = store.getAll();

      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }

  async addTask(task: Omit<Task, 'id'>): Promise<number> {
    await this.dbReady;
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(this.storeName, 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.add(task);

      request.onsuccess = () => resolve(request.result as number);
      request.onerror = () => reject(request.error);
    });
  }

  async updateTask(task: Task): Promise<void> {
    await this.dbReady;
    if (task.id === undefined) {
      throw new Error('Cannot update task without ID');
    }

    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(this.storeName, 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.put(task);

      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }

  async deleteTask(id: number): Promise<void> {
    await this.dbReady;
    return new Promise((resolve, reject) => {
      const transaction = this.db.transaction(this.storeName, 'readwrite');
      const store = transaction.objectStore(this.storeName);
      const request = store.delete(id);

      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
}

```

step8:C:\Users\Administrator\WebstormProjects\untitled4\src\app\student\student.component.html

```xml
<div class="task-manager">
  <div class="task-header">
    <input
      [(ngModel)]="newTask.title"
      placeholder="Enter task title"
      class="task-input"
      (keyup.enter)="addTask()"
    >
    <button
      class="primary-btn"
      (click)="addTask()"
    >
      ＋ Add Task
    </button>
  </div>

  <div class="task-list">
    <div *ngFor="let task of tasks" class="task-item">
      <input
        [(ngModel)]="task.title"
        class="task-input"
      >
      <select
        [(ngModel)]="task.status"
        (change)="updateTask(task)"
        class="status-select"
      >
        <option value="pending">⏳ Pending</option>
        <option value="completed">✅ Completed</option>
      </select>
      <button
        class="danger-btn"
        (click)="deleteTask(task.id!)"
      >
        🗑 Delete
      </button>
    </div>
  </div>
</div>

```

step9:C:\Users\Administrator\WebstormProjects\untitled4\src\app\student\student.component.css

```css
/* 现代配色方案 */
.task-manager {
  max-width: 800px;
  margin: 2rem auto;
  padding: 2rem;
  background: white;
  border-radius: 12px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.05);
}

.task-header {
  display: flex;
  gap: 1rem;
  margin-bottom: 2rem;
}

.task-input {
  flex: 1;
  padding: 12px;
  border: 2px solid #e9ecef;
  border-radius: 8px;
  font-size: 16px;
  transition: border-color 0.3s ease;
}
.task-input:focus {
  outline: none;
  border-color: #4a90e2;
  box-shadow: 0 0 0 3px rgba(74, 144, 226, 0.1);
}

.primary-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: linear-gradient(135deg, #4a90e2 0%, #2d7ff9 100%);
  color: white;
}
.primary-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  background: linear-gradient(135deg, #3d85d6 0%, #2770e0 100%);
}
.primary-btn:active {
  transform: translateY(0);
}

.danger-btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.3s ease;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  background: linear-gradient(135deg, #ff4757 0%, #ff6b81 100%);
  color: white;
}
.danger-btn:hover {
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
.danger-btn:active {
  transform: translateY(0);
}

.task-list {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.task-item {
  display: flex;
  gap: 1rem;
  align-items: center;
  padding: 1.5rem;
  background: #f8f9fa;
  border-radius: 8px;
  transition: transform 0.3s ease;
}
.task-item:hover {
  transform: translateX(5px);
}

.status-select {
  padding: 8px 12px;
  border: 2px solid #e9ecef;
  border-radius: 6px;
  background: white;
  cursor: pointer;
  transition: all 0.3s ease;
}
.status-select:focus {
  outline: none;
  border-color: #4a90e2;
}

```

 直接点击运行，然后success

end