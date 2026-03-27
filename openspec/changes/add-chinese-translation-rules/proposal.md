## Why

随着中文用户群体的增长，需要为项目中的文档和技能文件提供中文翻译。现有系统缺乏统一的翻译文件命名规范和翻译格式标准，导致翻译文件散落各处，格式不一致，难以维护和查找。

## What Changes

- 新增中文翻译文件命名约定：`{原文件名}.zh-CN.md`
- 新增翻译内容格式规范：保留原文，附加中文翻译的对照格式
- 定义 YAML frontmatter、标题、列表、代码注释、表格等不同元素的翻译规则

## Capabilities

### New Capabilities

- `chinese-translation-naming`: 定义翻译文件的命名规范，包括单文件和嵌套路径的处理方式
- `chinese-translation-format`: 定义翻译内容的格式规范，包括原文与译文的对照排版规则

### Modified Capabilities

- (无)

## Impact

- 影响范围：所有需要创建中文翻译的文档和技能文件
- 新增文件：翻译文件将按照约定命名并采用统一格式
- 无需修改现有代码，主要影响文档编写流程
