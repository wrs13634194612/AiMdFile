说明:
写一个简单的日历功能
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/0c67058d655e4309ab5c0b62577a84f8.png#pic_center)

step1:C:\Users\Administrator\WebstormProjects\untitled4\src\app\calendar\calendar.component.ts

```javascript
import { Component, signal } from '@angular/core';
import { CommonModule } from '@angular/common';
import { MatButtonModule } from '@angular/material/button';
import { MatIconModule } from '@angular/material/icon';
import { MatDialog, MatDialogModule } from '@angular/material/dialog';

interface CalendarDay {
  date: Date;
  isToday: boolean;
  isHoliday: boolean;
  events: string[];
  isWeekend: boolean;
}

@Component({
  selector: 'app-calendar',
  standalone: true,
  imports: [CommonModule, MatButtonModule, MatIconModule, MatDialogModule],
  templateUrl: './calendar.component.html',
  styleUrls: ['./calendar.component.css']
})
export class CalendarComponent {
  currentDate = signal(new Date());
  days = signal<CalendarDay[]>([]);
  darkMode = false;

  constructor(private dialog: MatDialog) {
    this.generateCalendar();
  }

  private generateCalendar() {
    const daysArray: CalendarDay[] = [];
    const year = this.currentDate().getFullYear();
    const month = this.currentDate().getMonth();
    const daysInMonth = new Date(year, month + 1, 0).getDate();

    for (let day = 1; day <= daysInMonth; day++) {
      const date = new Date(year, month, day);
      const dayOfWeek = date.getDay();

      daysArray.push({
        date,
        isToday: this.isSameDay(date, new Date()),
        isHoliday: this.checkHoliday(date),
        isWeekend: dayOfWeek === 0 || dayOfWeek === 6,
        events: []
      });
    }
    this.days.set(daysArray);
  }

  changeMonth(offset: number) {
    this.currentDate.update(d => new Date(d.getFullYear(), d.getMonth() + offset, 1));
    this.generateCalendar();
  }

  addEvent(day: CalendarDay) {
    // 事件添加逻辑（示例）
    day.events.push('New Event');
    this.days.update(days => [...days]);
  }

  private isSameDay(d1: Date, d2: Date) {
    return d1.toDateString() === d2.toDateString();
  }

  private checkHoliday(date: Date) {
    const holidays = [
      '2024-01-01', '2024-05-01', '2024-10-01', // 示例节假日
      '2024-02-10', '2024-04-04', '2024-06-10'
    ];
    const dateString = date.toISOString().split('T')[0];
    return holidays.includes(dateString);
  }
}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\calendar\calendar.component.html

```xml
<div class="calendar-container" [class.dark-mode]="darkMode">
  <div class="calendar-header">
    <button mat-icon-button (click)="changeMonth(-1)">
      <mat-icon>chevron_left</mat-icon>
    </button>

    <h2>{{ currentDate() | date: 'yyyy年MM月' }}</h2>

    <div class="control-buttons">
      <button mat-icon-button (click)="darkMode = !darkMode">
        <mat-icon>{{ darkMode ? 'dark_mode' : 'light_mode' }}</mat-icon>
      </button>
    </div>

    <button mat-icon-button (click)="changeMonth(1)">
      <mat-icon>chevron_right</mat-icon>
    </button>
  </div>

  <div class="calendar-grid">
    <div *ngFor="let day of days()"
         class="calendar-day"
         [class.today]="day.isToday"
         [class.holiday]="day.isHoliday"
         [class.weekend]="day.isWeekend"
         (click)="addEvent(day)">
      <div class="solar">{{ day.date | date: 'd' }}</div>
      <div class="weekday">{{ day.date | date: 'EEE' }}</div>
      <div class="events">
        <mat-icon *ngFor="let e of day.events">circle</mat-icon>
      </div>
    </div>
  </div>
</div>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\calendar\calendar.component.css

```css
.calendar-container {
  --bg-color: #ffffff;
  --text-color: #333;
  --header-bg: #f8f9fa;
  --border-color: #dee2e6;
  --today-bg: #e3f2fd;
  --holiday-bg: #ffebee;
  --weekend-bg: #f8f9fa;
  --event-color: #2196f3;
  --button-hover: rgba(0, 0, 0, 0.05);

  background: var(--bg-color);
  border-radius: 16px;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  transition: all 0.3s ease;
  max-width: 800px;
  margin: 2rem auto;
}

/* 暗黑模式 */
.calendar-container.dark-mode {
  --bg-color: #2d2d2d;
  --text-color: #e0e0e0;
  --header-bg: #1a1a1a;
  --border-color: #404040;
  --today-bg: #004d80;
  --holiday-bg: #4d1f1f;
  --weekend-bg: #333;
  --event-color: #64b5f6;
  --button-hover: rgba(255, 255, 255, 0.1);
}

.calendar-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px;
  background: var(--header-bg);
  border-radius: 12px;
  margin-bottom: 24px;

  h2 {
    margin: 0;
    color: var(--text-color);
    font-weight: 600;
    font-size: 1.5rem;
  }

  button {
    transition: all 0.2s ease;
    border-radius: 8px;

    &:hover {
      background: var(--button-hover);
    }

    mat-icon {
      color: var(--text-color);
    }
  }
}

.calendar-grid {
  display: grid;
  grid-template-columns: repeat(7, 1fr);
  gap: 8px;
}

.calendar-day {
  position: relative;
  min-height: 120px;
  padding: 12px;
  border: 1px solid var(--border-color);
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.2s ease;
  background: var(--bg-color);

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  }

  &.today {
    background: var(--today-bg);
    border-color: transparent;
  }

  &.holiday {
    background: var(--holiday-bg);
    border-color: transparent;

    .solar {
      color: #ff4444;
    }
  }

  &.weekend {
    background: var(--weekend-bg);

    .weekday {
      color: #888;
    }
  }
}

.solar {
  font-size: 1.4rem;
  font-weight: 600;
  color: var(--text-color);
}

.weekday {
  font-size: 0.9rem;
  color: var(--text-color);
  opacity: 0.8;
  margin: 4px 0;
}

.events {
  position: absolute;
  bottom: 8px;
  right: 8px;

  mat-icon {
    font-size: 16px;
    color: var(--event-color);
    margin-left: 2px;
  }
}

.control-buttons {
  display: flex;
  gap: 8px;
}

@media (max-width: 768px) {
  .calendar-container {
    padding: 16px;
  }

  .calendar-day {
    min-height: 90px;
    padding: 8px;

    .solar {
      font-size: 1.2rem;
    }
  }
}

```

end