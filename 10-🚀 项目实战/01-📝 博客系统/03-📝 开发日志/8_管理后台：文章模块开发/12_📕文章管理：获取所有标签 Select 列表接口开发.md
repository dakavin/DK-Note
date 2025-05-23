## 1 前言

接下来，我们将进入到页面中文章编辑功能的开发。在这之前呢，还遗漏了一个后端接口的开发工作 —— _获取所有标签 Select 列表接口_。大家知道，当用户点击编辑某篇文章按钮时，需要请求文章详情接口，先将数据渲染到页面中，然后用户才能修改，如下图所示：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/a258d0eccd94f95bd1379dc21a070901.png)

但是针对于文章标签这块，详情接口返回的是一个标签 `ID` 集合，但是只有 `ID` 还不够，我们还需要将其转换为标签名称，如下图所示，交互上才更加友好，不然用户都不知道当前文章打的什么标签：
![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/b28ddd3e227e23377d4fdc34da34db32.png)

想要达到上面的效果，就需要先请求 _获取所有标签 Select 列表接口_，此接口会返回所有标签，包括 `ID` 对应的名称，再搭配详情接口返回的标签 `ID` 集合，就知道哪些标签被选中了。

## 2 添加 service 业务层方法

接下来，我们着手开始添加该接口。首先，在 `AdminTagService` 业务接口中声明一个 _查询标签 Select 列表数据_ 方法，代码如下：

```java
public interface AdminTagService {

	// 省略...

    /**
     * 查询标签 Select 列表数据
     * @return
     */
    Response findTagSelectList();
}
```

然后在其 `AdminTagServiceImpl` 实现类中实现该方法，代码如下：

```java
    @Override 
    public Response findTagSelectList() {
        // 查询所有标签, Wrappers.emptyWrapper() 表示查询条件为空
        List<TagDO> tagDOS = tagMapper.selectList(Wrappers.emptyWrapper());

        // DO 转 VO
        List<SelectRspVO> vos = null;
        if (!CollectionUtils.isEmpty(tagDOS)) {
            vos = tagDOS.stream()
                    .map(tagDO -> SelectRspVO.builder()
                            .label(tagDO.getName())
                            .value(tagDO.getId())
                            .build())
                    .collect(Collectors.toList());
        }

        return Response.success(vos);
    }
```

上述代码中，我们首先查询出了标签表中所有的标签数据，然后对其进行判空，若不为空，则将 `DO` 类装换为 `SelectRspVO` 视图类，并返回给前端。

## 3 新增 controller 接口

业务层代码开发完成后，在 `AdminTagController` 控制器中，添加 `/admin/tag/select/list` 接口，代码如下:

```java
package com.quanxiaoha.weblog.admin.controller;

import com.quanxiaoha.weblog.admin.model.vo.tag.AddTagReqVO;
import com.quanxiaoha.weblog.admin.model.vo.tag.DeleteTagReqVO;
import com.quanxiaoha.weblog.admin.model.vo.tag.FindTagPageListReqVO;
import com.quanxiaoha.weblog.admin.model.vo.tag.SearchTagsReqVO;
import com.quanxiaoha.weblog.admin.service.AdminTagService;
import com.quanxiaoha.weblog.common.aspect.ApiOperationLog;
import com.quanxiaoha.weblog.common.utils.PageResponse;
import com.quanxiaoha.weblog.common.utils.Response;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.security.access.prepost.PreAuthorize;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/admin/tag")
@Api(tags = "Admin 标签模块")
public class AdminTagController {

    @Autowired
    private AdminTagService tagService;

    // 省略...

    @PostMapping("/select/list")
    @ApiOperation(value = "查询标签 Select 列表数据")
    @ApiOperationLog(description = "查询标签 Select 列表数据")
    public Response findTagSelectList() {
        return tagService.findTagSelectList();
    }

}
```

> 注意：由于之前 Controller 类上 `@RequestMapping` 注解中的接口前缀修改为了 `/admin/tag` ， 那么，方法级别的 `@PostMapping` 接口值无须再以 `/tag` 开头，这里需要注意一下，否则访问接口会报 404。

## 4 测试看看

重启项目，浏览器访问 `/doc.html` , 调试一波该接口，看看功能是否正常：

![image.png|600](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2024/05/3e03d0aa6af34a208a785e018d1d3bc3.png)

可以看到，该接口能够将标签数据以 `label` 名称和 `value` 值（ID） 的方式返回，接口测试通过。

## 5 结语

本小节中，我们将编辑文章所需的、之前遗漏的一个接口补上了 —— 获取所有标签 Select 列表。有了此接口，下小节中就来把文章编辑功能全部开发完成。