下面提供一份基于前面整理的影楼管理软件各个模块的数据库设计示例。下面的 SQL DDL 语句基于 MySQL 语法，每个表都包含主要字段及中文备注，方便你理解和后续开发维护。你可以根据实际业务需求调整和扩展字段。

---

### 1. 客户管理模块

```sql
CREATE TABLE Customer (
    CustomerID INT AUTO_INCREMENT PRIMARY KEY COMMENT '客户ID',
    Name VARCHAR(100) NOT NULL COMMENT '客户姓名',
    Phone VARCHAR(20) COMMENT '客户电话',
    Email VARCHAR(100) COMMENT '客户邮箱',
    Address VARCHAR(255) COMMENT '客户地址',
    CustomerType VARCHAR(50) COMMENT '客户类型（VIP、潜在、普通等）',
    Notes TEXT COMMENT '备注信息',
    CreateTime DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    UpdateTime DATETIME DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='客户管理表';
```

---

### 2. 预约与排期管理模块

```sql
CREATE TABLE Appointment (
    AppointmentID INT AUTO_INCREMENT PRIMARY KEY COMMENT '预约ID',
    CustomerID INT NOT NULL COMMENT '客户ID',
    AppointmentDate DATETIME NOT NULL COMMENT '预约时间',
    Location VARCHAR(255) COMMENT '拍摄地点',
    ServiceType VARCHAR(50) COMMENT '服务类型（婚纱、写真、儿童等）',
    Status VARCHAR(50) COMMENT '预约状态（待确认、已确认、取消）',
    CreateTime DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='预约与排期管理表';
```

---

### 3. 合同与订单管理模块

**合同管理表**

```sql
CREATE TABLE Contract (
    ContractID INT AUTO_INCREMENT PRIMARY KEY COMMENT '合同ID',
    CustomerID INT NOT NULL COMMENT '客户ID',
    ContractNumber VARCHAR(50) NOT NULL COMMENT '合同编号',
    ContractDate DATE COMMENT '签订日期',
    TotalAmount DECIMAL(10,2) COMMENT '总金额',
    Deposit DECIMAL(10,2) COMMENT '定金金额',
    Balance DECIMAL(10,2) COMMENT '尾款金额',
    Status VARCHAR(50) COMMENT '合同状态',
    Terms TEXT COMMENT '合同条款备注',
    CreateTime DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='合同管理表';
```

**订单管理表**

```sql
CREATE TABLE `Order` (
    OrderID INT AUTO_INCREMENT PRIMARY KEY COMMENT '订单ID',
    ContractID INT NOT NULL COMMENT '合同ID',
    OrderDate DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '下单日期',
    OrderStatus VARCHAR(50) COMMENT '订单状态（拍摄中、后期制作、已交付等）',
    Remarks TEXT COMMENT '订单备注',
    FOREIGN KEY (ContractID) REFERENCES Contract(ContractID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='订单管理表';
```

---

### 4. 拍摄管理模块

```sql
CREATE TABLE ShootingSession (
    SessionID INT AUTO_INCREMENT PRIMARY KEY COMMENT '拍摄记录ID',
    AppointmentID INT NOT NULL COMMENT '预约ID',
    PhotographerID INT COMMENT '摄影师ID',
    MakeupArtistID INT COMMENT '化妆师ID',
    Location VARCHAR(255) COMMENT '拍摄地点',
    StartTime DATETIME COMMENT '拍摄开始时间',
    EndTime DATETIME COMMENT '拍摄结束时间',
    Notes TEXT COMMENT '拍摄记录备注',
    FOREIGN KEY (AppointmentID) REFERENCES Appointment(AppointmentID)
    -- PhotographerID 和 MakeupArtistID 可作为外键关联员工表（Employee）
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='拍摄管理表';
```

---

### 5. 后期制作管理模块

```sql
CREATE TABLE PostProduction (
    ProductionID INT AUTO_INCREMENT PRIMARY KEY COMMENT '后期制作ID',
    OrderID INT NOT NULL COMMENT '订单ID',
    EditorID INT COMMENT '后期制作人员ID',
    Status VARCHAR(50) COMMENT '后期制作状态',
    StartTime DATETIME COMMENT '制作开始时间',
    EndTime DATETIME COMMENT '制作结束时间',
    Remarks TEXT COMMENT '制作备注',
    FOREIGN KEY (OrderID) REFERENCES `Order`(OrderID)
    -- EditorID 可关联员工表（Employee）
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='后期制作管理表';
```

---

### 6. 成品交付与售后服务模块

**成品交付表**

```sql
CREATE TABLE Delivery (
    DeliveryID INT AUTO_INCREMENT PRIMARY KEY COMMENT '交付ID',
    OrderID INT NOT NULL COMMENT '订单ID',
    DeliveryMethod VARCHAR(50) COMMENT '交付方式（线上下载、U盘、纸质相册等）',
    DeliveryDate DATETIME COMMENT '交付日期',
    CustomerFeedback TEXT COMMENT '客户反馈',
    FOREIGN KEY (OrderID) REFERENCES `Order`(OrderID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='成品交付表';
```

**售后服务表**

```sql
CREATE TABLE AfterSalesService (
    ServiceID INT AUTO_INCREMENT PRIMARY KEY COMMENT '售后服务ID',
    OrderID INT NOT NULL COMMENT '订单ID',
    ServiceType VARCHAR(50) COMMENT '服务类型（补拍、追加制作等）',
    RequestDate DATETIME COMMENT '服务申请日期',
    Resolution TEXT COMMENT '问题处理情况',
    FOREIGN KEY (OrderID) REFERENCES `Order`(OrderID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='售后服务管理表';
```

---

### 7. 财务管理模块

**财务交易记录表**

```sql
CREATE TABLE FinancialTransaction (
    TransactionID INT AUTO_INCREMENT PRIMARY KEY COMMENT '交易记录ID',
    OrderID INT COMMENT '订单ID',
    TransactionType VARCHAR(20) COMMENT '交易类型（收入/支出）',
    Amount DECIMAL(10,2) COMMENT '交易金额',
    TransactionDate DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '交易日期',
    Description TEXT COMMENT '交易描述',
    FOREIGN KEY (OrderID) REFERENCES `Order`(OrderID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='财务交易记录表';
```

**发票管理表**

```sql
CREATE TABLE Invoice (
    InvoiceID INT AUTO_INCREMENT PRIMARY KEY COMMENT '发票ID',
    OrderID INT NOT NULL COMMENT '订单ID',
    InvoiceNumber VARCHAR(50) COMMENT '发票编号',
    IssueDate DATE COMMENT '开票日期',
    Amount DECIMAL(10,2) COMMENT '发票金额',
    Status VARCHAR(50) COMMENT '发票状态',
    FOREIGN KEY (OrderID) REFERENCES `Order`(OrderID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='发票管理表';
```

---

### 8. 库存与设备管理模块

**库存管理表**

```sql
CREATE TABLE Inventory (
    InventoryID INT AUTO_INCREMENT PRIMARY KEY COMMENT '库存ID',
    ItemName VARCHAR(100) NOT NULL COMMENT '物品名称',
    Category VARCHAR(50) COMMENT '物品类别（服装、道具、化妆品等）',
    Quantity INT DEFAULT 0 COMMENT '库存数量',
    Location VARCHAR(100) COMMENT '存放位置',
    PurchaseDate DATE COMMENT '购买日期',
    Supplier VARCHAR(100) COMMENT '供应商',
    Status VARCHAR(50) COMMENT '库存状态（在库、缺货等）',
    Remarks TEXT COMMENT '备注信息'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='库存管理表';
```

**设备管理表**

```sql
CREATE TABLE Equipment (
    EquipmentID INT AUTO_INCREMENT PRIMARY KEY COMMENT '设备ID',
    EquipmentName VARCHAR(100) NOT NULL COMMENT '设备名称',
    Type VARCHAR(50) COMMENT '设备类型',
    PurchaseDate DATE COMMENT '购买日期',
    MaintenanceDate DATE COMMENT '上次维护日期',
    Status VARCHAR(50) COMMENT '设备状态（正常、维修中等）',
    Remarks TEXT COMMENT '设备备注'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='设备管理表';
```

---

### 9. 员工与人力资源管理模块

**员工信息表**

```sql
CREATE TABLE Employee (
    EmployeeID INT AUTO_INCREMENT PRIMARY KEY COMMENT '员工ID',
    FullName VARCHAR(100) NOT NULL COMMENT '员工姓名',
    Role VARCHAR(50) COMMENT '岗位/职位',
    Phone VARCHAR(20) COMMENT '联系电话',
    Email VARCHAR(100) COMMENT '电子邮箱',
    Address VARCHAR(255) COMMENT '联系地址',
    HireDate DATE COMMENT '入职日期',
    Salary DECIMAL(10,2) COMMENT '薪资',
    Remarks TEXT COMMENT '其他备注'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='员工信息表';
```

**员工考勤表**

```sql
CREATE TABLE Attendance (
    AttendanceID INT AUTO_INCREMENT PRIMARY KEY COMMENT '考勤记录ID',
    EmployeeID INT NOT NULL COMMENT '员工ID',
    Date DATE NOT NULL COMMENT '考勤日期',
    CheckInTime TIME COMMENT '签到时间',
    CheckOutTime TIME COMMENT '签退时间',
    Status VARCHAR(20) COMMENT '考勤状态（正常、迟到、缺勤等）',
    Remarks TEXT COMMENT '备注',
    FOREIGN KEY (EmployeeID) REFERENCES Employee(EmployeeID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='员工考勤表';
```

**绩效评估表**

```sql
CREATE TABLE PerformanceReview (
    ReviewID INT AUTO_INCREMENT PRIMARY KEY COMMENT '绩效评估ID',
    EmployeeID INT NOT NULL COMMENT '员工ID',
    ReviewDate DATE COMMENT '评估日期',
    Score DECIMAL(3,1) COMMENT '评分',
    Comments TEXT COMMENT '评估意见',
    FOREIGN KEY (EmployeeID) REFERENCES Employee(EmployeeID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='绩效评估表';
```

---

### 10. 数据统计与报表分析模块

（报表模块通常不直接存储原始业务数据，可选存储生成的统计数据）

```sql
CREATE TABLE Report (
    ReportID INT AUTO_INCREMENT PRIMARY KEY COMMENT '报表ID',
    ReportType VARCHAR(50) COMMENT '报表类型',
    GeneratedDate DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '生成日期',
    Data TEXT COMMENT '报表数据（可采用JSON格式存储）',
    Remarks TEXT COMMENT '备注'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='报表数据表';
```

---

### 11. 营销推广与CRM模块

**营销活动管理表**

```sql
CREATE TABLE MarketingCampaign (
    CampaignID INT AUTO_INCREMENT PRIMARY KEY COMMENT '营销活动ID',
    CampaignName VARCHAR(100) NOT NULL COMMENT '活动名称',
    StartDate DATE COMMENT '活动开始日期',
    EndDate DATE COMMENT '活动结束日期',
    Budget DECIMAL(10,2) COMMENT '活动预算',
    Description TEXT COMMENT '活动描述',
    Status VARCHAR(50) COMMENT '活动状态'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='营销活动管理表';
```

**客户互动记录表**

```sql
CREATE TABLE CRMInteraction (
    InteractionID INT AUTO_INCREMENT PRIMARY KEY COMMENT '客户互动记录ID',
    CustomerID INT NOT NULL COMMENT '客户ID',
    InteractionDate DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '互动日期',
    InteractionType VARCHAR(50) COMMENT '互动类型（电话、邮件、面谈等）',
    Notes TEXT COMMENT '互动内容记录',
    FOREIGN KEY (CustomerID) REFERENCES Customer(CustomerID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='客户互动记录表';
```

---

### 12. 系统管理与权限管理模块

**系统用户表**

```sql
CREATE TABLE UserAccount (
    UserID INT AUTO_INCREMENT PRIMARY KEY COMMENT '用户ID',
    Username VARCHAR(50) NOT NULL COMMENT '用户名',
    PasswordHash VARCHAR(255) NOT NULL COMMENT '密码哈希',
    Role VARCHAR(50) COMMENT '用户角色',
    Email VARCHAR(100) COMMENT '用户邮箱',
    CreatedAt DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统用户表';
```

**角色定义表**

```sql
CREATE TABLE RoleDefinition (
    RoleID INT AUTO_INCREMENT PRIMARY KEY COMMENT '角色ID',
    RoleName VARCHAR(50) NOT NULL COMMENT '角色名称',
    Description TEXT COMMENT '角色描述'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='角色定义表';
```

**权限表**

```sql
CREATE TABLE Permission (
    PermissionID INT AUTO_INCREMENT PRIMARY KEY COMMENT '权限ID',
    PermissionName VARCHAR(50) NOT NULL COMMENT '权限名称',
    Description TEXT COMMENT '权限描述'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='权限表';
```

**用户权限关联表**

```sql
CREATE TABLE UserPermission (
    UserID INT NOT NULL COMMENT '用户ID',
    PermissionID INT NOT NULL COMMENT '权限ID',
    PRIMARY KEY (UserID, PermissionID),
    FOREIGN KEY (UserID) REFERENCES UserAccount(UserID),
    FOREIGN KEY (PermissionID) REFERENCES Permission(PermissionID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户权限关联表';
```

**系统日志表**

```sql
CREATE TABLE SystemLog (
    LogID INT AUTO_INCREMENT PRIMARY KEY COMMENT '日志ID',
    UserID INT COMMENT '操作用户ID',
    Action VARCHAR(255) COMMENT '操作描述',
    Timestamp DATETIME DEFAULT CURRENT_TIMESTAMP COMMENT '操作时间',
    Details TEXT COMMENT '详细信息',
    FOREIGN KEY (UserID) REFERENCES UserAccount(UserID)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统操作日志表';
```

**系统配置表**

```sql
CREATE TABLE SystemConfig (
    ConfigID INT AUTO_INCREMENT PRIMARY KEY COMMENT '配置ID',
    ConfigKey VARCHAR(100) NOT NULL COMMENT '配置键',
    ConfigValue VARCHAR(255) COMMENT '配置值',
    Description TEXT COMMENT '配置描述'
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='系统配置表';
```

---

## 总结

以上数据库设计示例覆盖了影楼管理软件的主要业务模块，包括客户管理、预约、合同与订单、拍摄、后期制作、交付售后、财务、库存、设备、员工管理、数据报表、营销推广以及系统权限管理。每个表的字段都带有详细的中文备注，你可以根据实际需求对字段、数据类型或关系进行调整和扩展。这套 DB 设计旨在帮助你快速构建一个完整、清晰的影楼管理系统数据库。
