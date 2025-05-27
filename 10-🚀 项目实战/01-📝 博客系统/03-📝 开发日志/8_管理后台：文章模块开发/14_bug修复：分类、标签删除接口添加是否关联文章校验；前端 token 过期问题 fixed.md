---
文章标题: "[[14_bug修复：分类、标签删除接口添加是否关联文章校验；前端 token 过期问题 fixed]]" 
文章作者: Dakkk
文章概要: |
  文章主要修复了两类Bug：后端针对文章分类和标签删除操作，增加关联校验，防止脏数据产生；前端则处理了Token过期问题，当收到401状态码时，自动清除Token并跳转登录页，提升用户体验和系统健壮性。
文章标签:
- "Bug修复"
- "数据完整性"
- "后端开发"
- "前端开发"
- "认证鉴权"
- "异常处理"
- "SQL优化"
相关文章:
- "[[13_跨域]]"
- "[[11_📕全局异常处理器+参数校验（最佳实践）]]"
- "[[0_参考文章索引]]"
- "[[0_导读]]"
- "[[0_导论]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/8_管理后台：文章模块开发/14_bug修复：分类、标签删除接口添加是否关联文章校验；前端 token 过期问题 fixed.md"
文章难度: 初级 💧
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐⭐ 精通必备
创建时间: 2024-04-30 14:41:16
修改时间: 2025-05-27 00:58:15
---

本小节中，我们将完成文章管理模块的收尾工作，主要是对目前存在的一些 `Bug` 进行修复。如删除分类、标签数据时，若分类或者标签已经关联了文章，直接删除会导致脏数据的产生，如下图所示：

![](https://img.quanxiaoha.com/quanxiaoha/169769002216999)

直接删除文章对应得分类记录后，会导致渲染不出分类名称，只能展示一个分类 `ID` 了，显然是不合理的。标签也是同样的情况。所以，后端在进行删除的时候，需要对关联数据进行校验，如果已经关联了文章，则不允许删除，提示用户需要先删除分类、或标签下的所有文章，才能进行删除操作。

## 1. 添加分类删除校验

编辑 `ArticleCategoryRelMapper` 接口，为其封装一个 _根据分类 ID 查询_ 的默认方法，代码如下：

```
public interface ArticleCategoryRelMapper extends BaseMapper<ArticleCategoryRelDO> {

	// 省略...

    /**
     * 根据分类 ID 查询
     * @param categoryId
     * @return
     */
    default ArticleCategoryRelDO selectOneByCategoryId(Long categoryId) {
        return selectOne(Wrappers.<ArticleCategoryRelDO>lambdaQuery()
                .eq(ArticleCategoryRelDO::getCategoryId, categoryId)
                .last("LIMIT 1"));
    }
}
```

> **SQL 性能优化**：通过分类 `ID` 查询关联记录，从业务角度进行分析，当存在任何一条记录时，则都应该不允许删除。由于分类-文章是一对多的关系，也就是说会查询出多条数据，这里通过 `last()` 方法为 `SQL` 结尾添加 `LIMIT 1` , 保证只查询到第一条即可，可以提升查询性能，若不添加 `LIMIT 1` , 由于查询字段并没有添加唯一索引，`MYSQL` 还会继续往下查询，将所有属于该分类中的记录都查询出来。
> 
> ![](https://img.quanxiaoha.com/quanxiaoha/169768026463874)

接着回到 `AdminCategoryServiceImpl` 业务实现类中，编辑删除分类方法，为其添加关联数据校验逻辑，代码如下：

```
    @Autowired
    private ArticleCategoryRelMapper articleCategoryRelMapper;
    
    // 省略...
    
    @Override
    public Response deleteCategory(DeleteCategoryReqVO deleteCategoryReqVO) {
        // 分类 ID
        Long categoryId = deleteCategoryReqVO.getId();

        // 校验该分类下是否已经有文章，若有，则提示需要先删除分类下所有文章，才能删除
        ArticleCategoryRelDO articleCategoryRelDO = articleCategoryRelMapper.selectOneByCategoryId(categoryId);

        if (Objects.nonNull(articleCategoryRelDO)) {
            log.warn("==> 此分类下包含文章，无法删除，categoryId: {}", categoryId);
            throw new BizException(ResponseCodeEnum.CATEGORY_CAN_NOT_DELETE);
        }

        // 删除分类
        categoryMapper.deleteById(categoryId);

        return Response.success();
    }
```

当存在关联记录时，则手动抛出一条业务异常，提示用户：_该分类下包含文章，请先删除对应文章，才能删除！_ ， 否则执行正常的删除操作。另外，你还需要在全局异常枚举类中，添加以下枚举值：

```
CATEGORY_CAN_NOT_DELETE("20011", "该分类下包含文章，请先删除对应文章，才能删除！"),
```

回到分类管理页，再次删除已有关联文章的分类时，就会提示不允许操作了：

![](https://img.quanxiaoha.com/quanxiaoha/169768030819234)

## 2. 添加标签删除校验

同样的，删除标签业务逻辑中也要添加校验。首先，在 `ArticleTagRelMapper` 接口封装一个_根据标签 ID 查询_方法，代码如下：

```
public interface ArticleTagRelMapper extends InsertBatchMapper<ArticleTagRelDO> {

	// 省略...

    /**
     * 根据标签 ID 查询
     * @param tagId
     * @return
     */
    default ArticleTagRelDO selectOneByTagId(Long tagId) {
        return selectOne(Wrappers.<ArticleTagRelDO>lambdaQuery()
                .eq(ArticleTagRelDO::getTagId, tagId)
                .last("LIMIT 1"));
    }
}
```

然后，在其 `AdminTagServiceImpl` 业务实现类中，添加标签是否关联文章的校验逻辑，代码如下：

```
    @Autowired
    private ArticleTagRelMapper articleTagRelMapper;
    
    // 省略...
   
   /**
     * 删除标签
     *
     * @param deleteTagReqVO
     * @return
     */
    @Override
    public Response deleteTag(DeleteTagReqVO deleteTagReqVO) {
        // 标签 ID
        Long tagId = deleteTagReqVO.getId();

        // 校验该标签下是否有关联的文章，若有，则不允许删除，提示用户需要先删除标签下的文章
        ArticleTagRelDO articleTagRelDO = articleTagRelMapper.selectOneByTagId(tagId);

        if (Objects.nonNull(articleTagRelDO)) {
            log.warn("==> 此标签下包含文章，无法删除，tagId: {}", tagId);
            throw new BizException(ResponseCodeEnum.TAG_CAN_NOT_DELETE);
        }

        // 根据标签 ID 删除
        int count = tagMapper.deleteById(tagId);

        return count == 1 ? Response.success() : Response.fail(ResponseCodeEnum.TAG_NOT_EXISTED);
    }
```

全局枚举类中添加如下枚举值：

```
TAG_NOT_EXISTED("20007", "该标签不存在！"),
// 省略...
TAG_CAN_NOT_DELETE("20012", "该标签下包含文章，请先删除对应文章，才能删除！"),
```

## 3. token 过期自动跳转登录页

最后, 我们还要修复一下 `Token` 过期的问题，前端页面在会提示 `Token 已失效`， 就没有后续的动作了，需要我们手动退出登录再重新登陆，才能正常请求接口。正常是逻辑应该是，当前端请求接口后，应该进行判断，若后端返回 `401` 状态码时，应该清空 `Cookie` 中失效的 `Token` 令牌，自动跳转登录页，让用户重新登陆才对：

![](https://img.quanxiaoha.com/quanxiaoha/169768417674244)

为了模拟该问题，你可以手动将后端的 `tokenExpireTime` 令牌过期时间改为 `1` 分钟，等过期后回到页面中，再次请求任意 `/admin` 为前缀的接口，就会复现如上问题：

![](https://img.quanxiaoha.com/quanxiaoha/169768859449149)

_如何解决呢？_ 编辑前端工程中的 `axios.js` 文件，在响应拦截器中，拿到状态码 `status` , 若等于 _401_, 则手动清除 `Token` , 并刷新页面，代码如下：

```
import { removeToken } from '@/composables/cookie'

// 省略...

// 添加响应拦截器
instance.interceptors.response.use(function (response) {
    // 2xx 范围内的状态码都会触发该函数。
    // 对响应数据做点什么
    return response.data
}, function (error) {
    // 超出 2xx 范围的状态码都会触发该函数。
    // 对响应错误做点什么
    let status = error.response.status

    // 状态码 401
    if (status == 401) {
        // 删除 cookie 中的令牌
        removeToken()
        // 刷新页面
        location.reload()
    }

    // 若后台有错误提示就用提示文字，默认提示为 '请求失败'
    let errorMsg = error.response.data.message || '请求失败'
    // 弹错误提示
    showMessage(errorMsg, 'error')

    return Promise.reject(error)
})
```

刷新页面后，由于此时 `Cookie` 中的令牌已经被删除了，则会触发全局路由前置守卫中的逻辑判断，提示用户需先登录并跳转登录页，完成交互闭环：

![](https://img.quanxiaoha.com/quanxiaoha/169769156088642)

## 4. 结语

本小节中，我们主要完成了文章管理模块的收尾工作，包括分类、标签删除时，添加了是否已关联文章的校验，若是则不允许删除，并提示用户需先删除分类、标签下的所有文章，才可执行删除操作。最后，我们完善了前端交互，添加了当 `Token` 过期了可以自动跳转登录页功能。