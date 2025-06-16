---
文章标题: "[[5_📕文章分类：分类Select下拉列表接口开发]]"
文章作者: Dakkk
文章概要: |
  文章详细介绍了开发后台文章分类Select下拉列表接口。通过定义通用VO、实现Service层从数据库获取并转换数据，最终在Controller暴露API，为前端提供分类的`label`和`value`数据，供用户选择，功能实现简单直接。
文章标签:
  - API开发
  - Spring Boot
  - 后台管理
  - 下拉列表
  - RESTful API
  - Java
  - 分类管理
相关文章:
  - "[[16_📕获取当前登录用户信息接口开发]]"
  - "[[../../../../08-🛠️ 开发工具/03-🐋 容器化/01-📚 Docker基础/8_Docker Compose/3_实战web服务]]"
  - "[[1_📕分类管理模块接口分析]]"
  - "[[2_📕文章分类：新增接口开发]]"
  - "[[3_📕文章分类：分页接口开发]]"
文章分类: 🚀 项目实战
文章路径: 10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/5_管理后台：文章分类模块开发/5_📕文章分类：分类Select下拉列表接口开发.md
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-27 13:58:53
修改时间: 2025-05-27 00:56:23
---

本小节中，我们将后台管理中，和分类有关的最后一个接口 —— ==Select 下拉列表接口开发完成== ，后续在文章发布模块会用到此接口。

![img](https://img.quanxiaoha.com/quanxiaoha/169502690350643)img

如上图所示，当我们发布文章时，需要选择文章所属分类，点击分类 `Select` 下拉列表框时，前端会请求此接口获取所有分类数据，以供用户选择。

## 1 定义接口、出入参定义

### 1.1 接口地址

```
POST /admin/category/select/list
```

### 1.2 入参

_无入参。_ 因为是获取所有分类数据，所以并不需要筛选条件。

### 1.3 出参

```json
{
  "success": true,
  "message": null,
  "errorCode": null,
  "data": [
    {
      "label": "fdsfdsf", // select lable 展示文字
      "value": 5 // 对应的 ID
    },
    {
      "label": "小哈学Java",
      "value": 6
    },
    {
      "label": "犬小哈教程",
      "value": 7
    }
  ]
}
```

下拉 `Select` 列表通常需要两个字段值，`label` 用于文字展示，而 `value` 值则对应的是分类的 `ID` ，后续发布文章时，服务端判断文章归属分类时，就是根据此分类 `ID` 。

## 2 新建 Selelct 下拉列表的响应 VO

由于此接口没有入参，所以仅需要新增一个响应的 `VO` 类即可。然而，`Select` 下拉列表的数据格式属于比较通用的，比如获取文章标签 `Select` 数据，也是同样的数据格式。所以，完全可以封装一个通用的响应类。在 `weblog-module-common` 模块中的 `/model` 包下，新建一个 `/vo` 包，然后创建 `SelectRspVO` 通用响应类，代码如下：

![](https://img.quanxiaoha.com/quanxiaoha/169519972190559)

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
@Builder
public class SelectRspVO {
    /**
     * Select 下拉列表的展示文字
     */
    private String label;

    /**
     * Select 下拉列表的 value 值，如 ID 等
     */
    private Object value;
}
```

## 3 在 service 业务层添加获取 Select 列表方法

创建好响应 `VO` 类后，我们编辑 `AdminCategoryService` 接口，添加获取 `Select` 列表方法，代码如下：

```java
    /**
     * 获取文章分类的 Select 列表数据
     * @return
     */
    Response findCategorySelectList();
```

然后在 `AdminCategoryServiceImpl` 类中实现该方法：

```java
/**  
 * 创建文章时，在文章分类的下拉列表中展示分类的 文字描述 和 id  
 * @return */@Override  
public Response findCategorySelectList() {  
    // 查询所有分类  
    List<CategoryDO> categoryDOS = categoryMapper.selectList(null);  
      
    // DO 转 VO    List<SelectRspVO> selectRspVOS = null;  
    // 如果分类数据不为空  
    if (!CollectionUtils.isEmpty(categoryDOS)){  
        selectRspVOS = categoryDOS.stream()  
                .map(categoryDO -> SelectRspVO.builder()  
                        .label(categoryDO.getName())  
                        .value(categoryDO.getId())  
                        .build())  
                .collect(Collectors.toList());  
    }  
    return Response.success(selectRspVOS);  
}
```

上述方法中，我们直接使用 `categoryMapper` 内部封装的 `selectList()` 方法，不构造任何查询条件， 将 `null` 作为入参传入 , 也就是查询所有数据。然后，当查询到的分类数据不为空时，通过 `jdk 1.8` 的 `stream` 流，对数据库 `DO` 类转换成视图展示 `VO` 类，返回给前端 。注意，`label` 我们设置的是分类的名称，`value` 值则设置的分类 `ID` 。

## 4 在 controller 层添加接口

业务层代码编写完成后，我们在 `controller` 控制器层，添加 `/admin/category/select/list` 接口，代码如下：

```java
@PostMapping("/category/select/list")  
@ApiOperation("4-创建文章时，获取分类下拉列表中的数据")  
@ApiOperationLog(description = "创建文章时，获取分类下拉列表中的数据")  
public Response findCategorySelectList(){  
    return categoryService.findCategorySelectList();  
}
```

## 5 测试看看

重启项目，访问 `/doc.html` 文档页面，测试一波接口，看看功能是否正常。目前分类表中，小哈添加了 3 条分类记录，看看能否按照预想的格式返回：

![](https://img.quanxiaoha.com/quanxiaoha/169520045393063)

![](https://img.quanxiaoha.com/quanxiaoha/169519977288095)

可以看到接口返回的数据一切正常。
## 6 结语

本小结中，我们主要将获取分类 Select 下拉列表数据的接口搞定了，功能实现上很简单。后面小结中，我们将重回前端，将管理后台的文章分类管理模块开发完成。