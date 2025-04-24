说明：springboot+mysql增删改查

step1:create 

```bash
language: java

type: gradle-groovy

jdk:21

java:21

packaging: jar



developer tools:lombok

web:   spring web

template engines:thymelaf

sql:spring data jpa,spring data jdbc,mysql driver

i/o:validation

```

step2:sql

```sql
-- 商家表（独立存储商家信息）
CREATE TABLE db_school.merchants (
  merchant_id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  contact_phone VARCHAR(20) NOT NULL,
  address VARCHAR(255) NOT NULL,
  created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

```

step3:tree

```bash
src/
├── main/
│   ├── java/
│   │   └── com/
│   │       └── example/
│   │           └── demo/
│   │               ├── config/            # 配置类
│   │               │   └── SwaggerConfig.java
│   │               ├── controller/        # 控制器层
│   │               │   └── MerchantController.java
│   │               ├── dto/              # 数据传输对象
│   │               │   ├── MerchantDTO.java
│   │               │   └── ApiResponse.java
│   │               ├── exception/        # 异常处理
│   │               │   ├── GlobalExceptionHandler.java
│   │               │   └── ResourceNotFoundException.java
│   │               ├── model/            # 实体类
│   │               │   └── Merchant.java
│   │               ├── repository/      # 数据访问层
│   │               │   └── MerchantRepository.java
│   │               ├── service/         # 服务层
│   │               │   ├── MerchantService.java
│   │               │   └── impl/
│   │               │       └── MerchantServiceImpl.java
│   │               └── DemoApplication.java # 启动类
│   └── resources/
│       ├── static/       # 静态资源
│       ├── templates/   # 模板文件
│       └── application.yml # 配置文件
└── test/                # 测试目录
    └── java/
        └── com/
            └── example/
                └── demo/
                    ├── controller/
                    │   └── MerchantControllerTest.java
                    └── service/
                        └── MerchantServiceTest.java
```

step:4:data

```java
C:\Users\wangrusheng\IdeaProjects\demo10\src\main\java\com\example\demo\controller\MerchantController.java


package com.example.demo.controller;

import com.example.demo.exception.ResourceNotFoundException;
import com.example.demo.model.Merchant;
import com.example.demo.service.MerchantService;
import jakarta.validation.Valid;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.DeleteMapping;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.PutMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
import java.util.Collections;
import java.util.List;



@RestController
@RequestMapping("/api/merchants")
public class MerchantController {
    private final MerchantService merchantService;

    public MerchantController(MerchantService merchantService) {
        this.merchantService = merchantService;
    }

    @GetMapping
    public ResponseEntity<List<Merchant>> getAllMerchants() {
        return ResponseEntity.ok(merchantService.getAllMerchants());
    }

    @GetMapping("/{id}")
    public ResponseEntity<Merchant> getMerchantById(@PathVariable Integer id) {
        return ResponseEntity.ok(merchantService.getMerchantById(id));
    }

    @PostMapping
    public ResponseEntity<Merchant> createMerchant(@Valid @RequestBody Merchant merchant) {
        return new ResponseEntity<>(merchantService.createMerchant(merchant), HttpStatus.CREATED);
    }

    @PutMapping("/{id}")
    public ResponseEntity<Merchant> updateMerchant(
            @PathVariable Integer id,
            @Valid @RequestBody Merchant merchantDetails) {
        return ResponseEntity.ok(merchantService.updateMerchant(id, merchantDetails));
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<?> deleteMerchant(@PathVariable Integer id) {
        merchantService.deleteMerchant(id);
        return ResponseEntity.noContent().build();
    }

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<?> handleResourceNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(Collections.singletonMap("error", ex.getMessage()));
    }
}


C:\Users\wangrusheng\IdeaProjects\demo10\src\main\java\com\example\demo\exception\ResourceNotFoundException.java

package com.example.demo.exception;

public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String message) {
        super(message);
    }
}



C:\Users\wangrusheng\IdeaProjects\demo10\src\main\java\com\example\demo\model\Merchant.java


package com.example.demo.model;

import jakarta.persistence.Column;
import jakarta.persistence.Entity;
import jakarta.persistence.GeneratedValue;
import jakarta.persistence.GenerationType;
import jakarta.persistence.Id;
import jakarta.persistence.Table;
import java.time.LocalDateTime;
import org.hibernate.annotations.CreationTimestamp;

@Entity
@Table(name = "merchants")
public class Merchant {
    // ... 保持原有代码不变 ...

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer merchantId;

    @Column(nullable = false, length = 100)
    private String name;

    @Column(name = "contact_phone", nullable = false, length = 20)
    private String contactPhone;

    @Column(nullable = false, length = 255)
    private String address;

    @Column(updatable = false)
    @CreationTimestamp
    private LocalDateTime createdAt;

    // Getters and Setters


    public Integer getMerchantId() {
        return merchantId;
    }

    public void setMerchantId(Integer merchantId) {
        this.merchantId = merchantId;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getContactPhone() {
        return contactPhone;
    }

    public void setContactPhone(String contactPhone) {
        this.contactPhone = contactPhone;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public LocalDateTime getCreatedAt() {
        return createdAt;
    }

    public void setCreatedAt(LocalDateTime createdAt) {
        this.createdAt = createdAt;
    }
}


C:\Users\wangrusheng\IdeaProjects\demo10\src\main\java\com\example\demo\repository\MerchantRepository.java

package com.example.demo.repository;

import com.example.demo.model.Merchant;
import org.springframework.data.jpa.repository.JpaRepository;

public interface MerchantRepository extends JpaRepository<Merchant, Integer> {

}



C:\Users\wangrusheng\IdeaProjects\demo10\src\main\java\com\example\demo\service\MerchantService.java

package com.example.demo.service;

import com.example.demo.exception.ResourceNotFoundException;
import com.example.demo.model.Merchant;
import com.example.demo.repository.MerchantRepository;
import jakarta.transaction.Transactional;
import org.springframework.stereotype.Service;
import java.util.List;



@Service
public class MerchantService {
    private final MerchantRepository merchantRepository;

    public MerchantService(MerchantRepository merchantRepository) {
        this.merchantRepository = merchantRepository;
    }

    public List<Merchant> getAllMerchants() {
        return merchantRepository.findAll();
    }

    public Merchant getMerchantById(Integer id) {
        return merchantRepository.findById(id)
                .orElseThrow(() -> new ResourceNotFoundException("Merchant not found"));
    }

    public Merchant createMerchant(Merchant merchant) {
        return merchantRepository.save(merchant);
    }

    public Merchant updateMerchant(Integer id, Merchant merchantDetails) {
        Merchant merchant = getMerchantById(id);
        merchant.setName(merchantDetails.getName());
        merchant.setContactPhone(merchantDetails.getContactPhone());
        merchant.setAddress(merchantDetails.getAddress());
        return merchantRepository.save(merchant);
    }

    public void deleteMerchant(Integer id) {
        Merchant merchant = getMerchantById(id);
        merchantRepository.delete(merchant);
    }
}


C:\Users\wangrusheng\IdeaProjects\demo10\src\main\resources\application.properties

# 数据库连接配置
spring.datasource.url=jdbc:mysql://localhost:3306/db_school?useSSL=false
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
# 服务端口配置
server.port=5200



```

step5:postman

```bash
 1. 获取所有商户列表   http://localhost:5200/api/merchants

 [
    {
        "merchantId": 1,
        "name": "Chow Tai Fook Jewelry",
        "contactPhone": "13800000001",
        "address": "No.1 Wangfujing Street, Beijing",
        "createdAt": "2025-03-19T14:48:40"
    },
    {
        "merchantId": 2,
        "name": "Lao Feng Xiang Silver Store",
        "contactPhone": "13800000002",
        "address": "No.100 Nanjing Road, Shanghai",
        "createdAt": "2025-03-19T14:48:40"
    },
    {
        "merchantId": 3,
        "name": "Lukfook Jewelry",
        "contactPhone": "13800000003",
        "address": "Zhujiang New Town, Tianhe District, Guangzhou",
        "createdAt": "2025-03-19T14:48:40"
    },
    {
        "merchantId": 4,
        "name": "Chow Sang Sang",
        "contactPhone": "13800000004",
        "address": "CBD, Futian District, Shenzhen",
        "createdAt": "2025-03-19T14:48:40"
    },
    {
        "merchantId": 5,
        "name": "Tse Sui Luen",
        "contactPhone": "13800000005",
        "address": "Intime Department Store, West Lake District, Hangzhou",
        "createdAt": "2025-03-19T14:48:40"
    }
]

2. 创建新商户 http://localhost:5200/api/merchants

{
    "name": "Hehe Lukfook Jewelry Flagship Store",
    "contactPhone": "13511112222",
    "address": "Zhujiang New Town, Tianhe District, Guangzhou"
}


{
    "merchantId": 11,
    "name": "Hehe Lukfook Jewelry Flagship Store",
    "contactPhone": "13511112222",
    "address": "Zhujiang New Town, Tianhe District, Guangzhou",
    "createdAt": "2025-03-20T10:00:56.122415Z"
}

3. 查询单个商户


  http://localhost:5200/api/merchants/11

{
    "merchantId": 11,
    "name": "Hehe Lukfook Jewelry Flagship Store",
    "contactPhone": "13511112222",
    "address": "Zhujiang New Town, Tianhe District, Guangzhou",
    "createdAt": "2025-03-20T10:00:56Z"
}

4. 更新商户信息

  http://localhost:5200/api/merchants/11
{
    "name": "Dage Jewelry Flagship Store",
    "contactPhone": "13511112222",
    "address": "Zhujiang New Town, Tianhe District, Guangzhou"
}

{
    "merchantId": 11,
    "name": "Dage Jewelry Flagship Store",
    "contactPhone": "13511112222",
    "address": "Zhujiang New Town, Tianhe District, Guangzhou",
    "createdAt": "2025-03-20T10:00:56Z"
}

5.删除商户信息
  http://localhost:5200/api/merchants/11

  测试删除成功
```

end