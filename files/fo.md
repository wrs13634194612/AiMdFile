说明:
使用angular实现表格，包括数据源，排序，分页
效果图：
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/3a2d0dab631440f1bbebfaf27eef9d29.png#pic_center)

step1:C:\Users\Administrator\WebstormProjects\untitled4\src\app\setting\setting.component.ts

```javascript
import { Component, ViewChild, AfterViewInit } from '@angular/core';
import { MatTableModule, MatTableDataSource } from '@angular/material/table';
import { MatPaginator, MatPaginatorModule } from '@angular/material/paginator';
import { MatSort, MatSortModule } from '@angular/material/sort';
import { CommonModule } from '@angular/common';
import { MatInputModule } from '@angular/material/input';
import { MatFormFieldModule } from '@angular/material/form-field';
import {MatIcon} from '@angular/material/icon';

export interface UserData {
  id: string;
  name: string;
  email: string;
  role: 'Admin' | 'User' | 'Guest';
}

@Component({
  selector: 'app-setting',
  standalone: true,
  imports: [
    CommonModule,
    MatTableModule,
    MatPaginatorModule,
    MatSortModule,
    MatInputModule,
    MatFormFieldModule,
    MatIcon
  ],
  templateUrl: './setting.component.html',
  styleUrls: ['./setting.component.css']
})
export class SettingComponent implements AfterViewInit {
  displayedColumns: string[] = ['id', 'name', 'email', 'role'];
  dataSource: MatTableDataSource<UserData>;

  @ViewChild(MatPaginator) paginator!: MatPaginator;
  @ViewChild(MatSort) sort!: MatSort;

  constructor() {
    // 生成测试数据
    const users = Array.from({ length: 100 }, (_, i) => ({
      id: (i + 1).toString(),
      name: `User_${i + 1}`,
      email: `user${i + 1}@demo.com`,
      role: this.getRole(i)
    }));
    this.dataSource = new MatTableDataSource<UserData>(users);
  }

  private getRole(index: number): 'Admin' | 'User' | 'Guest' {
    if (index % 3 === 0) return 'Admin';
    if (index % 2 === 0) return 'User';
    return 'Guest';
  }

  ngAfterViewInit() {
    this.dataSource.paginator = this.paginator;
    this.dataSource.sort = this.sort;
  }

  applyFilter(event: Event) {
    const filterValue = (event.target as HTMLInputElement).value;
    this.dataSource.filter = filterValue.trim().toLowerCase();

    if (this.dataSource.paginator) {
      this.dataSource.paginator.firstPage();
    }
  }
}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\setting\setting.component.html

```xml
<div class="container">
  <!-- 搜索框 -->
  <mat-form-field appearance="outline">
    <mat-label>Filter users</mat-label>
    <input
      matInput
      (keyup)="applyFilter($event)"
      placeholder="Search..."
      data-testid="search-input"
    >
    <mat-icon matSuffix>search</mat-icon>
  </mat-form-field>

  <!-- 数据表格 -->
  <table
    mat-table
    [dataSource]="dataSource"
    matSort
    class="mat-elevation-z8"
  >
    <!-- ID列 -->
    <ng-container matColumnDef="id">
      <th mat-header-cell *matHeaderCellDef mat-sort-header>ID</th>
      <td mat-cell *matCellDef="let user">{{ user.id }}</td>
    </ng-container>

    <!-- 姓名列 -->
    <ng-container matColumnDef="name">
      <th mat-header-cell *matHeaderCellDef mat-sort-header>Name</th>
      <td mat-cell *matCellDef="let user">{{ user.name }}</td>
    </ng-container>

    <!-- 邮箱列 -->
    <ng-container matColumnDef="email">
      <th mat-header-cell *matHeaderCellDef mat-sort-header>Email</th>
      <td mat-cell *matCellDef="let user">{{ user.email }}</td>
    </ng-container>

    <!-- 角色列 -->
    <ng-container matColumnDef="role">
      <th mat-header-cell *matHeaderCellDef mat-sort-header>Role</th>
      <td mat-cell *matCellDef="let user">
        <span [class]="'role-badge ' + user.role.toLowerCase()">
          {{ user.role }}
        </span>
      </td>
    </ng-container>

    <!-- 表头行 -->
    <tr mat-header-row *matHeaderRowDef="displayedColumns"></tr>
    <!-- 数据行 -->
    <tr
      mat-row
      *matRowDef="let row; columns: displayedColumns"
      class="row-hover"
    ></tr>
  </table>

  <!-- 分页器 -->
  <mat-paginator
    [pageSizeOptions]="[5, 10, 20]"
    showFirstLastButtons
    aria-label="Select page of users"
  ></mat-paginator>
</div>

```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\setting\setting.component.css

```css
.container {
  padding: 20px;
  max-width: 1200px;
  margin: 0 auto;
}

mat-form-field {
  width: 100%;
  max-width: 400px;
  margin-bottom: 20px;
}

table {
  width: 100%;
  margin: 20px 0;
}

.role-badge {
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 0.8em;
  font-weight: 500;
}

.role-badge.admin {
  background-color: #f0f8ff;
  color: #1e90ff;
}

.role-badge.user {
  background-color: #f0fff0;
  color: #228b22;
}

.role-badge.guest {
  background-color: #fff8dc;
  color: #daa520;
}

.row-hover:hover {
  background-color: #f5f5f5;
  cursor: pointer;
}

.mat-elevation-z8 {
  box-shadow: 0 5px 5px -3px rgba(0,0,0,.2),
  0 8px 10px 1px rgba(0,0,0,.14),
  0 3px 14px 2px rgba(0,0,0,.12);
}

```

end