---
文章标题: "[[9_smart composer插件使用]]" 
文章作者: Dakkk
文章概要: |
  本文简要指导 Smart Composer 插件的 embedding 参数优化，重点调整 Chunk size、Threshold tokens、Minimum similarity 和 Limit，以提升大语言模型（如 Gemini）的 RAG 效果，并强调了 API 限制和索引重建的重要性。
文章标签:
- "Smart Composer"
- "Embedding"
- "RAG"
- "参数调优"
- "大语言模型"
- "向量搜索"
- "Gemini"
相关文章:
- "[[12_数据库其它调优策略]]"
- "[[7_搭建本地ai知识库]]"
文章分类: "🔧 系统配置"
文章路径: "13-🔧 系统配置/05-🛠️ Obsidian技巧/9_smart composer插件使用.md"
文章难度: 中级 🌳
目前阶段: ✅ 已完成
重要性: ⭐⭐⭐ 重要技能
创建时间: 2025-05-26 04:02:47
修改时间: 2025-05-26 04:03:57
---

## 1 调整embedding参数

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/653c54594c06a496a8e234273eb7eb47.png)

![image.png|500](https://my-obsidian-image.oss-cn-guangzhou.aliyuncs.com/2025/05/8b368432a6ac25063ea9206a29dc9b1d.png)

**Chunk size**: `2048`
- 接近 Gemini 的输入令牌限制 (2048)，最大化每个块的信息密度
**Threshold tokens**: `2048`
- 与 chunk size 保持一致，避免因令牌限制导致的 RAG 切换
**Minimum similarity**: `0.1-0.3`
- 由于 Gemini 的输出维度较高 (768)，可以使用较低的相似度阈值获得更多相关结果
**Limit**: `8-15`
- 考虑到每分钟 1500 个请求的限制，适度增加结果数量以提高上下文质量

1. **API 限制**: 每分钟 1500 个请求，如果文档较多需要考虑并发控制
2. **向量维度**: 768 维输出提供了良好的语义表示
3. **重建索引**: 更改 chunk size 后需要重建 embedding 数据库