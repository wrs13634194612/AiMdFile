用vue写时区选择器 

1.纽约   巴黎 伦敦 东京  首尔 迪拜  新德里  里约热内卢 北京   

2.默认显示北京时间   

3.展示日期和时间

4.用select选择器，选择对应的时区  分别显示北京和对应时区的日期和时间

5.可以手动选择 日期 和时间  然后点击按钮，监听用户选择的select时区， 分别显示北京和对应时区的日期和时间

效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/02be1e5b70cd41588be581b236e92554.png#pic_center)

step1:vue  C:\Users\wangrusheng\PycharmProjects\untitled3\src\components\HelloWorld.vue

```typescript
<template>
  <div>
    <div>
      <label>选择时区：</label>
      <select v-model="selectedTimezone">
        <option v-for="tz in timeZones" :key="tz.value" :value="tz.value">
          {{ tz.name }}
        </option>
      </select>
    </div>

    <div>
      <label>手动选择时间：</label>
      <input
        type="datetime-local"
        v-model="manualDateTime"
      />
      <button @click="handleManualTime">更新时间</button>
    </div>

    <div>
      <h3>北京时间：</h3>
      <p>{{ beijingTime }}</p>
    </div>

    <div>
      <h3>{{ selectedZoneName }}时间：</h3>
      <p>{{ selectedZoneTime }}</p>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      timeZones: [
        { name: "纽约", value: "America/New_York" },
        { name: "巴黎", value: "Europe/Paris" },
        { name: "伦敦", value: "Europe/London" },
        { name: "东京", value: "Asia/Tokyo" },
        { name: "首尔", value: "Asia/Seoul" },
        { name: "迪拜", value: "Asia/Dubai" },
        { name: "新德里", value: "Asia/Kolkata" },
        { name: "里约热内卢", value: "America/Sao_Paulo" },
        { name: "北京", value: "Asia/Shanghai" }
      ],
      selectedTimezone: "Asia/Shanghai",
      beijingTime: "",
      selectedZoneTime: "",
      manualDateTime: ""
    };
  },
  computed: {
    selectedZoneName() {
      return this.timeZones.find(tz => tz.value === this.selectedTimezone)?.name || "";
    }
  },
  mounted() {
    this.updateTime();
    setInterval(() => this.updateTime(), 1000);
  },
  methods: {
    updateTime() {
      const now = new Date();
      this.beijingTime = this.formatTime(now, "Asia/Shanghai");
      this.selectedZoneTime = this.formatTime(now, this.selectedTimezone);
    },
    handleManualTime() {
      if (!this.manualDateTime) {
        alert("请选择日期和时间");
        return;
      }
      const parts = this.manualDateTime.split(/[-T:]/);
      const year = parseInt(parts[0]);
      const month = parseInt(parts[1]);
      const day = parseInt(parts[2]);
      const hour = parseInt(parts[3]);
      const minute = parseInt(parts[4]);

      // 创建基于UTC的Date对象（假设输入时间为选中时区的本地时间）
      const date = new Date(Date.UTC(year, month - 1, day, hour, minute));

      // 获取选中时区在该时间的偏移量（分钟）
      const offset = this.getTimezoneOffset(this.selectedTimezone, date);

      // 调整UTC时间
      date.setUTCMinutes(date.getUTCMinutes() - offset);

      // 更新显示时间
      this.beijingTime = this.formatTime(date, "Asia/Shanghai");
      this.selectedZoneTime = this.formatTime(date, this.selectedTimezone);
    },
    formatTime(date, timeZone) {
      return new Intl.DateTimeFormat("zh-CN", {
        timeZone,
        dateStyle: "full",
        timeStyle: "long",
        hour12: false
      }).format(date);
    },
    getTimezoneOffset(timeZone, date) {
      const formatter = new Intl.DateTimeFormat("en-US", {
        timeZone,
        timeZoneName: "shortOffset"
      });
      const parts = formatter.formatToParts(date);
      const tzPart = parts.find(p => p.type === "timeZoneName");
      const offsetString = tzPart.value.replace(/GMT/g, "");

      let hours, minutes;
      if (offsetString.includes(":")) {
        [hours, minutes] = offsetString.split(":").map(Number);
      } else {
        hours = parseInt(offsetString);
        minutes = 0;
      }
      return hours * 60 + minutes; // 返回总分钟数（考虑正负）
    }
  }
};
</script>
```

step2:angular  C:\Users\wangrusheng\PycharmProjects\untitled\src\app\timezone\timezone.component.ts

```typescript
import { Component, OnInit, OnDestroy } from '@angular/core';
import {FormsModule} from '@angular/forms';
import {NgForOf} from '@angular/common';

@Component({
  selector: 'app-timezone',
  imports: [
    FormsModule,
    NgForOf
  ],
  templateUrl: './timezone.component.html',
  styleUrl: './timezone.component.css'
})
export class TimezoneComponent implements OnInit, OnDestroy {

timeZones = [
    { name: "纽约", value: "America/New_York" },
    { name: "巴黎", value: "Europe/Paris" },
    { name: "伦敦", value: "Europe/London" },
    { name: "东京", value: "Asia/Tokyo" },
    { name: "首尔", value: "Asia/Seoul" },
    { name: "迪拜", value: "Asia/Dubai" },
    { name: "新德里", value: "Asia/Kolkata" },
    { name: "里约热内卢", value: "America/Sao_Paulo" },
    { name: "北京", value: "Asia/Shanghai" }
  ];

  selectedTimezone = "Asia/Shanghai";
  beijingTime = "";
  selectedZoneTime = "";
  manualDateTime = "";
  private intervalId: any;

  ngOnInit() {
    this.updateTime();
    this.intervalId = setInterval(() => this.updateTime(), 1000);
  }

  ngOnDestroy() {
    if (this.intervalId) {
      clearInterval(this.intervalId);
    }
  }

  get selectedZoneName(): string {
    return this.timeZones.find(tz => tz.value === this.selectedTimezone)?.name || "";
  }

  updateTime() {
    const now = new Date();
    this.beijingTime = this.formatTime(now, "Asia/Shanghai");
    this.selectedZoneTime = this.formatTime(now, this.selectedTimezone);
  }

  handleManualTime() {
    if (!this.manualDateTime) {
      alert("请选择日期和时间");
      return;
    }

    const parts = this.manualDateTime.split(/[-T:]/);
    const year = parseInt(parts[0]);
    const month = parseInt(parts[1]);
    const day = parseInt(parts[2]);
    const hour = parseInt(parts[3]);
    const minute = parseInt(parts[4]);

    const date = new Date(Date.UTC(year, month - 1, day, hour, minute));
    const offset = this.getTimezoneOffset(this.selectedTimezone, date);

    date.setUTCMinutes(date.getUTCMinutes() - offset);

    this.beijingTime = this.formatTime(date, "Asia/Shanghai");
    this.selectedZoneTime = this.formatTime(date, this.selectedTimezone);
  }

  private formatTime(date: Date, timeZone: string): string {
    return new Intl.DateTimeFormat("zh-CN", {
      timeZone,
      dateStyle: "full",
      timeStyle: "long",
      hour12: false
    }).format(date);
  }

  private getTimezoneOffset(timeZone: string, date: Date): number {
    const formatter = new Intl.DateTimeFormat("en-US", {
      timeZone,
      timeZoneName: "shortOffset"
    });

    const parts = formatter.formatToParts(date);
    const tzPart = parts.find(p => p.type === "timeZoneName");
    const offsetString = tzPart?.value.replace(/GMT/g, "") || "+0";

    let hours, minutes;
    if (offsetString.includes(":")) {
      [hours, minutes] = offsetString.split(":").map(Number);
    } else {
      hours = parseInt(offsetString);
      minutes = 0;
    }

    return hours * 60 + (hours < 0 ? -minutes : minutes);
  }
}

<div>
  <div class="control-group">
    <label>选择时区：</label>
    <select [(ngModel)]="selectedTimezone">
      <option *ngFor="let tz of timeZones" [value]="tz.value">
        {{ tz.name }}
      </option>
    </select>
  </div>

  <div class="control-group">
    <label>手动选择时间：</label>
    <input
      type="datetime-local"
      [(ngModel)]="manualDateTime"
    />
    <button (click)="handleManualTime()">更新时间</button>
  </div>

  <div class="time-display">
    <h3>北京时间：</h3>
    <p>{{ beijingTime }}</p>
  </div>

  <div class="time-display">
    <h3>{{ selectedZoneName }}时间：</h3>
    <p>{{ selectedZoneTime }}</p>
  </div>
</div>

.control-group {
  margin-bottom: 20px;
  padding: 10px;
  background-color: #f5f5f5;
  border-radius: 4px;
}

.control-group label {
  margin-right: 10px;
}

select, input[type="datetime-local"] {
  padding: 5px;
  margin-right: 10px;
}

button {
  padding: 5px 15px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #0056b3;
}

.time-display {
  margin: 20px 0;
  padding: 15px;
  background-color: #fff;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.time-display h3 {
  color: #333;
  margin-bottom: 10px;
}

.time-display p {
  font-size: 1.2em;
  color: #666;
}

```

end