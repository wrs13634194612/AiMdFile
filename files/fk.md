说明:登录 login+dialog
效果图：

step1:

```javascript
import { Component } from '@angular/core';
import {FormGroup, FormControl, Validators, FormsModule, ReactiveFormsModule} from '@angular/forms';
import { MatDialog } from '@angular/material/dialog';
import { AlertDialogComponent } from './alert-dialog.component';
import {NgIf} from '@angular/common';

@Component({
  selector: 'app-login',
  templateUrl: './login.component.html',
  imports: [
    FormsModule,
    NgIf,
    ReactiveFormsModule
  ],
  styleUrls: ['./login.component.css']
})
export class LoginComponent {
  loginForm = new FormGroup({
    username: new FormControl('', [
      Validators.required,
      Validators.minLength(3)
    ]),
    password: new FormControl('', [
      Validators.required,
      Validators.minLength(6)
    ])
  });

  constructor(private dialog: MatDialog) {}

  get username() { return this.loginForm.get('username'); }
  get password() { return this.loginForm.get('password'); }

  onSubmit() {
    if (this.loginForm.invalid) return;

    const { username, password } = this.loginForm.value;

    if (username === 'admin' && password === '123456') {
      this.showAlert('登录成功', 'success');
      this.loginForm.reset();
    } else {
      this.showAlert('用户名或密码错误', 'error');
    }
  }

  private showAlert(message: string, type: 'success' | 'error') {
    this.dialog.open(AlertDialogComponent, {
      data: { message, type },
      width: '300px'
    });
  }
}

```

step2:

```xml
<form [formGroup]="loginForm" (ngSubmit)="onSubmit()" class="login-form">
  <h2>用户登录</h2>

  <div class="form-group">
    <label for="username">用户名</label>
    <input
      type="text"
      id="username"
      formControlName="username"
      placeholder="至少3个字符"
      [class.invalid]="username?.invalid"
    >
    <div class="error-message" *ngIf="username?.errors?.['required'] && username?.touched">
      用户名不能为空
    </div>
    <div class="error-message" *ngIf="username?.errors?.['minlength'] && username?.touched">
      用户名至少3个字符
    </div>
  </div>

  <div class="form-group">
    <label for="password">密码</label>
    <input
      type="password"
      id="password"
      formControlName="password"
      placeholder="至少6位密码"
      [class.invalid]="password?.invalid"
    >
    <div class="error-message" *ngIf="password?.errors?.['required'] && password?.touched">
      密码不能为空
    </div>
    <div class="error-message" *ngIf="password?.errors?.['minlength'] && password?.touched">
      密码至少6个字符
    </div>
  </div>

  <button
    type="submit"
    class="submit-btn"
    [disabled]="loginForm.invalid"
  >
    立即登录
  </button>
</form>

```

step3:

```css
.login-form {
  max-width: 300px;
  margin: 0 auto;
  padding: 20px;
}

.form-group {
  margin-bottom: 15px;
}

.form-group label {
  display: block;
  margin-bottom: 5px;
}

.form-group input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

.form-group input.invalid {
  border-color: #dc3545;
}

.error-message {
  color: #dc3545;
  font-size: 0.875rem;
  margin-top: 4px;
}

.submit-btn {
  width: 100%;
  padding: 10px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.submit-btn:hover {
  background-color: #0056b3;
}

```

step4:

```javascript
import { Component, Inject } from '@angular/core';
import { MAT_DIALOG_DATA, MatDialogModule, MatDialogRef } from '@angular/material/dialog';
import { NgClass } from '@angular/common';
import {MatButton} from "@angular/material/button";

@Component({
  selector: 'app-alert-dialog',
  standalone: true,
  imports: [MatDialogModule, NgClass, MatButton],
  templateUrl: './alert-dialog.component.html',
  styleUrls: ['./alert-dialog.component.css']
})
export class AlertDialogComponent {
  constructor(
    @Inject(MAT_DIALOG_DATA) public data: { message: string; type: 'success' | 'error' },
    public dialogRef: MatDialogRef<AlertDialogComponent>
  ) {}
}

```

step5:

```xml
<div class="alert-dialog" [ngClass]="data.type">
  <div class="alert-content">
    <h3 class="alert-title">
      <span class="alert-icon">  {{ data.type === 'success' ? '✅' : '❌' }} {{ data.message }} </span>
    </h3>
    <div class="action-buttons">
      <button mat-button class="alert-action" (click)="dialogRef.close()">
        确定
      </button>
    </div>
  </div>
</div>

```

step6:

```css
.alert-dialog {
  border-radius: 16px;
  box-shadow: 0 8px 24px rgba(0, 0, 0, 0.12);
  padding: 24px;
  background: #fff;
  margin: 0 16px;
}

.alert-title {
  margin: 0 0 16px 0;
  font-size: 18px;
  font-weight: 600;
  color: rgba(0, 0, 0, 0.87);
}

.alert-icon {
  margin-right: 8px;
  font-size: 20px;
}

.action-buttons {
  display: flex;
  justify-content: flex-end;
  gap: 8px;
}

.alert-action {
  border-radius: 8px !important;
  padding: 8px 16px !important;
  font-size: 14px !important;
  font-weight: 500 !important;
  transition: all 0.3s ease;
}

.alert-dialog[ng-reflect-ng-class="success"] {
  background: #e8f5e9;
}

.alert-dialog[ng-reflect-ng-class="success"] .alert-title,
.alert-dialog[ng-reflect-ng-class="success"] .alert-action {
  color: #2e7d32;
}

.alert-dialog[ng-reflect-ng-class="error"] {
  background: #ffebee;
}

.alert-dialog[ng-reflect-ng-class="error"] .alert-title,
.alert-dialog[ng-reflect-ng-class="error"] .alert-action {
  color: #c62828;
}

```
end