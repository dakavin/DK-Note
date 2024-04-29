
参考链接：https://worktile.com/kb/p/47075
## 1 来源不同

Autowired 和 Resource 注解来自不同的“父类”，其中**Autowired注解**是 Spring 定义的注解，而**Resource 注解**是 Java 定义的注解，它来自于 JSR-250（Java 250 规范提案）。

## 2 注入规则不同

**Autowired注解**是spring的注解,此注解只根据type进行注入,不会去匹配name.但是如果只根据type无法辨别注入对象时,就需要配合使用@Qualifier注解或者@Primary注解使用。

**Resource注解**有两个重要的属性，分别是name和type，如果name属性有值，则使用byName的自动注入策略，将值作为需要注入bean的名字，如果type有值，则使用byType自动注入策略，将值作为需要注入bean的类型。如果既不指定name也不指定type属性，这时将通过反射机制使用byName自动注入策略。即@Resource注解默认按照名称进行匹配，名称可以通过name属性进行指定，如果没有指定name属性，当注解写在字段上时，默认取字段名，按照名称查找,当找不到与名称匹配的bean时才按照类型进行装配。但是需要注意的是，如果name属性一旦指定，就只会按照名称进行装配。

## 3 依赖查找的顺序不同

- **Autowired注解**先根据类型（byType）查找，如果存在多个（Bean）再根据名称（byName）进行查找；
- **Resource注解**先根据名称（byName）查找，如果（根据名称）查找不到，再根据类型（byType）进行查找。

## 4 支持的参数不同

**Autowired注解**只支持设置 1 个参数，而**Resource注解**支持设置 7 个参数。

## 5 依赖注入的用法支持不同

**Autowired注解**支持属性注入、构造方法注入和 Setter 注入，而**Resource注解**只支持属性注入和 Setter 注入。

## 6 编译器 IDEA 的提示不同

当使用 IDEA 专业版在编写依赖注入的代码时，如果注入的是 Mapper 对象，那么使用**Autowired注解**编译器会提示报错信息。虽然 IDEA 会出现报错信息，但程序是可以正常执行的。 然后，我们再将依赖注入的注解更改为**Resource注解**就不会出现报错信息了

## 7 使用位置不同

两者都可以写在字段和setter方法上，如果写在字段上，那么就不需要在写setter方法。推荐使用**Resource注解**在字段上，这样不仅不需要写setter方法了，而且由于**Resource注解**属于J2EE，降低与spring的耦合。

## 8 共同点

1. @Resource注解和@Autowired注解都可以用作bean的注入；
2. 在接口只有一个实现类的时候,两个注解可以互相替换,效果相同。