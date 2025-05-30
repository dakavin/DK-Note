---
文章标题: "[[1_properties 和 yml]]" 
文章作者: Dakkk
文章概要: |
  文章对比了`.yml` (YAML)和`.properties`两种配置文件格式。YAML支持复杂数据结构和缩进，可读性强，适用于复杂配置；Properties为Java原生，简单键值对，适用于简单场景。文章分析了二者差异及在Spring Boot中的选择建议。
文章标签:
- "YAML"
- "Properties"
- "配置文件"
- "数据结构"
- "配置管理"
- "Spring Boot"
- "格式比较"
相关文章:
- "[[7_Springboot]]"
- "[[0_参考文章索引]]"
- "[[0_导读]]"
- "[[0_二叉树]]"
- "[[0_哈希表理论基础]]"
文章分类: "🚀 项目实战"
文章路径: "10-🚀 项目实战/01-📝 博客系统/03-📝 开发日志/0_知识补充/1_properties 和 yml.md"
文章难度: 入门 🌱
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐⭐ 核心能力
创建时间: 2024-04-17 00:47:41
修改时间: 2024-04-21 15:57:11
---


`.yml`（或`.yaml`）和`.properties`都是用来存储配置信息的文件格式，分别代表了 YAML（YAML Ain't Markup Language）和 Properties 两种格式。

**YAML（.yml 或 .yaml）**

YAML 是一种人类可读的数据序列化标准，通常用于配置文件和在网络上交换数据，如 API 交互等。YAML 以数据为中心，采用缩进式层次结构，从而使得配置文件更加清晰，易于阅读和书写。此外，YAML 支持复杂的数据结构，如列表和键值对，因此，==YAML 的使用适用于需要表示复杂数据结构的场合==。

一个简单的 YAML 文件示例：
```yml
person: 
	name: John Doe 
	age: 30 
	hobbies: 
		- Reading 
		- Sports
```

**Properties（.properties）**

Properties 格式是 Java 平台原生的一种配置文件格式。它的格式简单，每一行都是一个键值对，键和值之间用等号（=）或冒号（:）分隔。Properties 文件不支持复杂的数据结构，只支持字符串类型的键值对，因此，它的==使用适用于简单的配置场景==。

一个简单的 Properties 文件示例：
```properties
person.name=John Doe 
person.age=30
```

**YAML vs Properties**

- 可读性：YAML 的可读性更强，它采用缩进表示层次关系，而 Properties 文件则通过点分隔符表示层次关系。
- 数据结构：YAML 支持复杂的数据结构，包括列表和映射，而 Properties 文件只支持字符串类型的键值对。
- 兼容性：Properties 是 Java 平台的标准配置格式，几乎所有的 Java 程序都可以直接读取。而 YAML 需要相应的解析库才能解析。

在 Spring Boot 应用中，你可以根据自己的实际需求，选择使用 YAML 还是 Properties 格式的配置文件。如果你的配置比较简单，Properties 格式可能会更好些。如果你的配置比较复杂，或者你希望配置文件更具可读性，那么 YAML 格式可能是更好的选择。`blog` 项目选择使用 `.yml` 格式。