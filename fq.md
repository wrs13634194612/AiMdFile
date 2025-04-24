说明
angular 实现路由效果 router
效果图:

step0:

```bash
ng generate component user

ng generate component Home

```

step1:C:\Users\Administrator\WebstormProjects\untitled4\angular.json

```bash
         {
                "glob": "**/*",
                "input": "public"
              }
            ],
            "styles": [
              "src/styles.css"
            ],
            "scripts": []
          }
        }
      }
    }
  },
  "cli": {
    "analytics": "d5b895fd-4811-413d-bf05-8c0e8301e191"
  }
}

```

step2: C:\Users\Administrator\WebstormProjects\untitled4\src\app\app.component.html

```html
<h1>主页启动页</h1>
<nav>
  <a routerLink="/home">首页</a>
  <a routerLink="/user">个人中心</a>
</nav>
<router-outlet></router-outlet>
```

step3: C:\Users\Administrator\WebstormProjects\untitled4\src\app\app.component.ts

```javascript
import {RouterLink, RouterOutlet} from '@angular/router';

@Component({
  selector: 'app-root',
  standalone: true,
  imports: [RouterOutlet, RouterLink],
```


step4: C:\Users\Administrator\WebstormProjects\untitled4\src\app\app.routes.ts

```javascript

import { Routes } from '@angular/router';
import { HomeComponent} from './home/home.component';
import { UserComponent} from './user/user.component';

export const routes: Routes = [
  {
    path: 'home',
    component: HomeComponent,
  },
  {
    path: 'user',
    component: UserComponent
  }
];


```

end