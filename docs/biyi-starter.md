#biyi-starter 快速开发框架开发指引
## 概述

biyi-starter 快速开发矿建是基于 JPA 设计的一系列基础类。业务工程只需根据 JPA 规范设计业务实体类（Entity），并继承或实现基础的Entity、Repository、Service、Controller 等类和接口即可完成 CRUD 接口的开发。查询接口支持动态查询条件，可以在不更新代码的情况下实现一些复杂的条件查询。查询接口也支持预设条件查询，可以通过提交简单的参数实现条件查询。

## 特性

- 标准 JPA
- 级联查询
- 级联保存
- 级联删除
- 动态条件查询
- 预设条件查询

## maven 依赖

```xml
<dependency>
    <groupId>com.play.cloudframework</groupId>
    <artifactId>biyi-starter</artifactId>
</dependency>
```

## 基础类介绍

### 基础实体类

#### BiyiBaseEntity.java

基础实体类，所有业务实体类（Entity）都继承此类，此类中定义了以下属性

| 属性  | 数据类型 | 说明  |
| --- | --- | --- |
| createBy | String | 创建者userName |
| createTime | Date | 创建时间 |
| updateBy | String | 更新者userName |
| updateTime | Date | 更新时间 |
| lockFlag | Integer | 锁定标识 |
| delFlag | Integer | 逻辑删除标识 |

#### 示例

```java
@Data
@EqualsAndHashCode(callSuper = false)
@Entity
@Table(name = "demo_parent")
public class DemoParent extends BiyiBaseEntity implements Serializable {

  private static final long serialVersionUID = 1L;

  @Id
  @GeneratedValue(generator = "biyi")
  private String id;

  private String name;

}

```

### JPA 仓库接口

#### BiyiJpaRepository.java

仓库接口，该接口继承了 Jpa 实现需要的接口，业务仓库接口继承此接口

#### 示例

```java
@Repository
public interface DemoParentRepository extends BiyiJpaRepository<DemoParent, String> {

}
```

### JPA 服务接口

#### BiyiJpaInterfaceService.java

服务接口，该接口实现了 CRUD 的核心逻辑，业务服务接口继承此接口，业务类实现接口定义的一些方法

#### 示例

```java
public interface DemoParentService extends BiyiJpaInterfaceService<DemoParent, String> {

}
```

```java
@Service
public class DemoParentServiceImpl
    implements DemoParentService {

  @Autowired
  private DemoParentRepository demoParentRepository;

  @Override
  public BiyiJpaRepository<DemoParent, String> getRepository() {
    return demoParentRepository;
  }

}
```

### JPA Rest 控制类

#### BiyiJpaRestController.java

基础接口类，该类实现了以下 CRUD 接口

| 接口  | 请求方法 | 说明  |
| --- | --- | --- |
| /page | POST | 条件分页查询，支持动态条件查询 |
| /list | POST | 条件列表查询，支持动态条件查询 |
| /{id} | GET | 查询单条数据 |
| /   | POST | 添加  |
| /{id} | PUT | 修改  |
| /{id} | DELETE | 删除  |
| /import | POST | 导入  |
| /export | POST | 导出  |

#### 示例

```java
@RestController
@RequestMapping("/demo/parent")
@Api(value = "demo_parent", tags = "演示父级")
public class DemoParentController extends BiyiJpaRestController<DemoParent, String> {

  @Resource
  private DemoParentService demoParentService;

  @Override
  public BiyiJpaInterfaceService<DemoParent, String> getService() {
    return demoParentService;
  }

}
```

### JPA 预设条件 Rest 控制类

#### BiyiJpaPreConditionRestController.java

预设条件接口类，该类实现了支持预设查询条件的接口

| 接口  | 请求方法 | 说明  |
| --- | --- | --- |
| /page | GET | 条件分页查询，支持动态条件查询 |
| /list | GET |     |

#### 示例

```java
@RestController
@RequestMapping("/demo/parent")
@Api(value = "demo_parent", tags = "演示父级")
public class DemoParentController
    extends BiyiJpaPreConditionRestController<DemoParent, String, DemoParentParamDTO> {

  @Resource
  private DemoParentService demoParentService;

  @Override
  public BiyiJpaInterfaceService<DemoParent, String> getService() {
    return demoParentService;
  }

}
```

#### 前置条件

定义 DemoParentParamDTO.java

## 动态查询

### 请求接口方式

```http
POST xxx/page 
```

```http
POST xxx/list
```

### 动态查询语法

动态查询语法为 JSON 的一种格式，接口调用者通过构建对应格式的 JSON 即可实现对应的条件查询，语法格式如下

```json
{
    "property": "value"
}
```

```json
{
    "property": {
      "expression": "value"
    }
}
```

### 示例

```json
{
  "name": "张三",
  "age": {
    "lt": 29
  }
}
```

查询条件为查询`name`为`"张三"`并且`age`小于`29`的数据

备注，所有条件之间为`与`的关系

### 查询表达式支持如下

| 表达式 | 支持数据类型 | 说明  |
| --- | --- | --- |
| 无   | 基础数据类型（与属性定义的数据类型一致） | 等值比较 |
| min | 数值、字符串，日期（字符串，格式为"2023-01-01"和"2023-01-01 23:59:59"）（与属性定义的数据类型一致） | 区间最小值，默认为闭区间，通过minClose=false，设置为开区间，如果没有max条件，该条件等价于gt或gte |
| max | 数值、字符串，日期（字符串，格式为"2023-01-01"和"2023-01-01 23:59:59"）（与属性定义的数据类型一致） | 区间最大值，默认为闭区间，通过maxClose=false，设置为开区间，如果没有min条件，该条件等价于lt或lte |
| lt  | 数值、字符串，日期（字符串，格式为"2023-01-01"和"2023-01-01 23:59:59"）（与属性定义的数据类型一致） | 小于  |
| lte | 数值、字符串，日期（字符串，格式为"2023-01-01"和"2023-01-01 23:59:59"）（与属性定义的数据类型一致） | 小于或等于 |
| gt  | 数值、字符串，日期（字符串，格式为"2023-01-01"和"2023-01-01 23:59:59"）（与属性定义的数据类型一致） | 大于  |
| gte | 数值、字符串，日期（字符串，格式为"2023-01-01"和"2023-01-01 23:59:59"）（与属性定义的数据类型一致） | 大于或等于 |
| in  | 数组，数组元素为基础数据类型（与属性定义的数据类型一致） | 多个等值比较条件 |
| contain | 字符串 | 包含该字符串，等价于"%xxx%" |
| startWith | 字符串 | 以该字符串开始，等价于"xxx%" |
| endWith | 字符串 | 以该字符串结束，等价于"%xxx" |

### 分页排序

分页排序参数是在分页查询接口中可以设置的参数，示例如下

```json
{
  "pageNumber": 1,
  "pageSize": 20,
  "orders": [
    {
      "property": "id",
      "asc": true
    }
  ],
  "params": {
    "name": "张三",
    "age": {
      "lt": 29
    },
    "dataScope": ["admin"]
  }
}
```

## 预设条件查询

### 请求接口方式

```http
GET xxx/page
```

```http
GET xxx/list
```

### 预设条件参数类

查询参数类需要继承`BiyiBaseParamDTO`类，并在静态块中调用`register()`方法，示例如下

```java
@Data
@EqualsAndHashCode(callSuper = false)
public class DemoParentParamDTO extends BiyiBaseParamDTO<DemoParent> {

  static {
    register(DemoParentParamDTO.class, DemoParent::getName, ConditionTypeEnum.CONTAIN,
        DemoParentParamDTO::getName);
    register(DemoParentParamDTO.class, DemoParent::getCreateTime, ConditionTypeEnum.LT,
        DemoParentParamDTO::getDateStr);
  }

  private String name;

  private String dateStr;

}
```

### 预设条件查询语法

时间格式为`yyyy-MM-dd`

```http
GET xxx/list?name=xxx&dateStr=2023-01-01
```

查询条件为查询`name包含`"xxx"并且`createDate`小于`2023-01-01`的数据

或如下时间格式，`yyyy-MM-dd HH:mm:ss`

```http
GET xxx/list?name=xxx&dateStr=2023-01-01%2023%3A59%3A59
```

### 分页排序

```http
GET xxx/page?pageNumber=1&pageSize=20&order=createDate%20asc&name=xxx&dateStr=2023-01-01
```

order格式为`<property> asc[desc]`或者`<property>`，当order不指定排序方式时，排序方式为desc

## JPA相关

### 主键生成器

在`com.play.webframework.demo`或`com.play.webframework.demo.entity`包下面配置package-info.java

```java
@GenericGenerator(name = "biyi",
    strategy = "com.play.cloudframework.starter.domain.persistence.BiyiUUIDGenerator")
package com.play.webframework.demo;

import org.hibernate.annotations.GenericGenerator;
```

或

```java
@GenericGenerator(name = "biyi",
    strategy = "com.play.cloudframework.starter.domain.persistence.BiyiUUIDGenerator")
package com.play.webframework.demo.entity;

import org.hibernate.annotations.GenericGenerator;

```

完成以上配置后，可以在业务实体类中使用主键生成器，示例如下

```java
package com.play.webframework.demo.entity;

import java.io.Serializable;
import javax.persistence.Entity;
import javax.persistence.Table;
import com.play.cloudframework.starter.control.entity.BiyiBaseEntity;
import lombok.Data;
import lombok.EqualsAndHashCode;

@Data
@EqualsAndHashCode(callSuper = false)
@Entity
@Table(name = "demo_entity")
public class DemoEntity extends BiyiBaseEntity implements Serializable {

  private static final long serialVersionUID = 1L;

  @Id
  @GeneratedValue(generator = "biyi")
  private String id;

}
```

### 设置级联关系

在接口`BiyiCascade`中声明了`setCascadeRelationship`方法

```java
public interface BiyiCascade {

  default void setCascadeRelationship() {}

}
```

该方法在业务实体对象保存和更新之前会调用，开发者通过实现该方法来设置业务实体间的关系，从而达到级联保存的效果，示例如下

DemoParent.java

```java
package com.play.webframework.demo.entity;

import java.io.Serializable;
import java.util.Collections;
import java.util.List;
import java.util.Optional;
import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.JoinColumn;
import javax.persistence.JoinTable;
import javax.persistence.ManyToMany;
import javax.persistence.OneToMany;
import javax.persistence.Table;
import com.play.cloudframework.starter.control.entity.BiyiBaseEntity;
import lombok.Data;
import lombok.EqualsAndHashCode;

@Data
@EqualsAndHashCode(callSuper = false)
@Entity
@Table(name = "demo_parent")
public class DemoParent extends BiyiBaseEntity implements Serializable {

  private static final long serialVersionUID = 1L;

  @Id
  @GeneratedValue(generator = "biyi")
  private String id;

  private String name;

  @OneToMany(mappedBy = "parent", cascade = CascadeType.ALL, orphanRemoval = true)
  private List<DemoChild> children;

  @ManyToMany(cascade = CascadeType.ALL)
  @JoinTable(name = "demo_address_parent", joinColumns = {@JoinColumn(name = "parent_id")},
      inverseJoinColumns = {@JoinColumn(name = "address_id")})
  private List<DemoAddress> addresses;

  @Override
  public void setCascadeRelationship() {
    Optional.ofNullable(this.children)
        .ifPresent(children -> children.forEach(e -> e.setParent(this)));
    Optional.ofNullable(this.addresses).ifPresent(
        children -> children.forEach(e -> e.setParents(Collections.singletonList(this))));
  }

}
```

DemoChild.java

```java
package com.play.webframework.demo.entity;

import java.io.Serializable;
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.FetchType;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToOne;
import javax.persistence.Table;
import com.play.cloudframework.starter.control.entity.BiyiBaseEntity;
import lombok.Data;
import lombok.EqualsAndHashCode;

@Data
@EqualsAndHashCode(callSuper = false)
@Entity
@Table(name = "demo_child")
public class DemoChild extends BiyiBaseEntity implements Serializable {

  private static final long serialVersionUID = 1L;

  @Id
  @GeneratedValue(generator = "biyi")
  private String id;

  private String name;

  @Column(name = "parent_id", insertable = false, updatable = false)
  private String parentId;

  @ManyToOne(fetch = FetchType.LAZY)
  private DemoParent parent;

}
```

DemoAddress.java

```java
package com.play.webframework.demo.entity;

import java.io.Serializable;
import java.util.List;
import javax.persistence.CascadeType;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.Id;
import javax.persistence.ManyToMany;
import javax.persistence.Table;
import com.play.cloudframework.starter.control.entity.BiyiBaseEntity;
import lombok.Data;
import lombok.EqualsAndHashCode;

@Data
@EqualsAndHashCode(callSuper = false)
@Entity
@Table(name = "demo_address")
public class DemoAddress extends BiyiBaseEntity implements Serializable {

  private static final long serialVersionUID = 1L;

  @Id
  @GeneratedValue(generator = "biyi")
  private String id;

  private String name;

  @ManyToMany(mappedBy = "addresses", cascade = CascadeType.ALL)
  private List<DemoParent> parents;

}
```

