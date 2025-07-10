
## 1 of_find_node_by_path

文件目录：`include/linux/of.h`

```C
static inline struct device_node *of_find_node_by_path(const char *path)
{
        return of_find_node_opts_by_path(path, NULL);
}
```

查找与完整路径节点名称匹配的节点。

- 通过别名搜索时，无需以’`/'`字符开头。
    

## 2 of_find_node_opts_by_path

文件:`drivers/of/base.c`

```C
/**
 *      of_find_node_opts_by_path - Find a node matching a full OF path
 *      @path: Either the full path to match, or if the path does not
 *             start with '/', the name of a property of the /aliases
 *             node (an alias).  In the case of an alias, the node
 *             matching the alias' value will be returned.
 *      @opts: Address of a pointer into which to store the start of
 *             an options string appended to the end of the path with
 *             a ':' separator.
 *  *      Valid paths:
 *              /foo/bar        Full path
 *              foo             Valid alias
 *              foo/bar         Valid alias + relative path
 *      
 *      Returns a node pointer with refcount incremented, use
 *      of_node_put() on it when done.
 */     
struct device_node *of_find_node_opts_by_path(const char *path, const char **opts)
{
        struct device_node *np = NULL;
        struct property *pp;
        unsigned long flags;
        const char *separator = strchr(path, ':');

        if (opts)
                *opts = separator ? separator + 1 : NULL; 
                                             
        if (strcmp(path, "/") == 0)
                return of_node_get(of_root);

        /* The path could begin with an alias */
        if (*path != '/') {
                int len;
                const char *p = separator;

                if (!p)
                        p = strchrnul(path, '/');
                len = p - path;

                /* of_aliases must not be NULL */
                if (!of_aliases)         
                        return NULL;
     
                for_each_property_of_node(of_aliases, pp) {
                        if (strlen(pp->name) == len && !strncmp(pp->name, path, len)) {
                                np = of_find_node_by_path(pp->value);
                                break;
                        }
                }
                if (!np)
                        return NULL;
                path = p;
        }
```

查找与完整路径或别名匹配的节点。如果有一个可选的字符串（“`：`”之后的字符串），则输出参数将保存（如果给定的话）。

- 代码`23`行：查找路径中“`：`”字符的位置。
    
- 代码`25~26`行：如果有一个’`：`‘字符，则选择指向’`：`'之后的下一个字符，如果没有，则分配空值。
    
- 代码`28~29`行：如果是根节点，则返回全局变量`of_root`的节点。 如果节点名称是别名，则如果添加到全局`of_alias`的所有属性的值都相同，那么将使用该属性值来检索节点。
    
- 代码`32`行：如果节点名称以别名类型开头
    
- 代码`36~37`行：如果未指定`p`，则将’`/`'字符串的位置分配给`p`。
    
- 代码`38`行：紧凑别名的长度是从’/'字符串位置减去属性的第一个字符串位置的数目。
    
- 代码`44`行：遍历`of_aliases`节点中的所有属性
    
- 代码`45~48`行：如果属性名称长度和len（别名名称长度）相同，则将属性名称和别名的长度进行比较，如果长度相同，则通过查找具有属性值的节点来逃避循环。
    

```C
        /* Step down the tree matching path components */
        raw_spin_lock_irqsave(&devtree_lock, flags);
        if (!np)
                np = of_node_get(of_root);
        while (np && *path == '/') {
                path++; /* Increment past '/' delimiter */
                np = __of_find_node_by_path(np, path);
                path = strchrnul(path, '/');
                if (separator && separator < path)
                        break;
        }
        raw_spin_unlock_irqrestore(&devtree_lock, flags);
        return np;
}
EXPORT_SYMBOL(of_find_node_opts_by_path);
```

- 代码`3~4`行：如果没有找到节点信息，则检索根节点信息。
    
- 代码`5`行：在节点中，只有完整路径节点名称（而不是别名）运行循环。
    
- 代码`7`行：通过除前面的“ `/`”之外的完整路径节点名称搜索该节点。
    
- 代码`8`行：在完整路径节点名称中找到第二个“`/`”字符。
    
- 代码`9`行：如果第二个’`/`'字符串的位置大于分隔符，则退出循环并返回。
    

## 3 `__of_find_node_by_path`

![image.png|500|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/06/7c583497f8f5c17a5f0a49ba1582a49a.png)

```C
static struct device_node *__of_find_node_by_path(struct device_node *parent,
                                                const char *path)
{
        struct device_node *child;
        int len;

        len = strcspn(path, "/:");
        if (!len)
                return NULL;

        __for_each_child_of_node(parent, child) {
                const char *name = strrchr(child->full_name, '/');
                if (WARN(!name, "malformed device_node %s\n", child->full_name))
                        continue;
                name++;
                if (strncmp(path, name, len) == 0 && (strlen(name) == len))
                        return child;
        }
        return NULL;
}
```

将子节点的压缩节点名称与请求的字符串进行比较，如果子节点相同，则返回子节点；如果未找到，则返回`null`。请求字符串不能以’`/`'字符开头。

## 4 `__for_each_child_of_node`

查找节点的子节点。如果`prev`为`null`，则搜索第一个子节点；如果`prev`是子节点，则搜索下一个节点。

```C
#define __for_each_child_of_node(parent, child) \
        for (child = __of_get_next_child(parent, NULL); child != NULL; \
             child = __of_get_next_child(parent, child))
```

## 5 `__of_get_next_child`

```C
static struct device_node *__of_get_next_child(const struct device_node *node,
                                                struct device_node *prev)
{
        struct device_node *next;

        if (!node)
                return NULL;

        next = prev ? prev->sibling : node->child;
        for (; next; next = next->sibling)
                if (of_node_get(next))
                        break;
        of_node_put(prev);
        return next;
}
```

## 6 of_node_get

增加节点的参考计数器。

```C
/**
 * of_node_get() - Increment refcount of a node
 * @node:       Node to inc refcount, NULL is supported to simplify writing of
 *              callers
 *
 * Returns node.
 */
struct device_node *of_node_get(struct device_node *node)
{
        if (node)
                kobject_get(&node->kobj);
        return node;
}
EXPORT_SYMBOL(of_node_get);
```