# Monolith 翻译功能使用指南

## 概述

monolith 已成功集成 translation-lib 翻译库，现在支持对网页内容进行自动翻译。该功能可以将任何语言的网页内容翻译为中文，同时保持网页的完整结构和样式。

## 功能特性

- ✅ **智能文本识别**: 自动识别需要翻译的文本内容
- ✅ **代码块保护**: 自动跳过代码块、样式表等不需要翻译的内容
- ✅ **属性翻译**: 翻译 title、alt、placeholder 等属性中的文本
- ✅ **异步并行处理**: 支持高效的并行翻译处理
- ✅ **错误容忍**: 翻译失败时保持原文本，不中断整个流程
- ✅ **网络请求优化**: 内置速率限制，避免API过载

## 编译选项

翻译功能作为可选特性，需要在编译时启用：

```bash
# 启用翻译功能编译
cargo build --features translation

# 或者同时启用多个特性
cargo build --features "cli,translation,vendored-openssl"
```

## 使用方法

### 基本用法

```bash
# 启用翻译功能，使用默认设置
monolith https://example.com -T -o translated.html
```

### 指定目标语言

```bash
# 翻译为中文
monolith https://example.com -T --target-lang zh -o translated.html

# 翻译为英文
monolith https://example.com -T --target-lang en -o translated.html

# 翻译为日文
monolith https://example.com -T --target-lang ja -o translated.html
```

### 指定翻译API

```bash
# 使用自定义翻译API
monolith https://example.com -T \
  --target-lang zh \
  --translation-api "https://your-api.com/translate" \
  -o translated.html
```

### 结合其他选项

```bash
# 翻译 + 隔离模式 + 无JavaScript
monolith https://example.com -T -I -j --target-lang zh -o translated.html

# 翻译 + 静默模式
monolith https://example.com -T -q --target-lang zh -o translated.html
```

## 命令行参数

### 翻译相关参数

| 参数 | 短参数 | 说明 | 默认值 |
|------|--------|------|--------|
| `--translate` | `-T` | 启用翻译功能 | 禁用 |
| `--target-lang` | - | 目标语言代码 | `zh` |
| `--translation-api` | - | 翻译API地址 | DeepLX默认API |

### 支持的语言代码

| 语言 | 代码 | 示例 |
|------|------|------|
| 中文 | `zh` | `--target-lang zh` |
| 英文 | `en` | `--target-lang en` |
| 日文 | `ja` | `--target-lang ja` |
| 法文 | `fr` | `--target-lang fr` |
| 德文 | `de` | `--target-lang de` |
| 西班牙文 | `es` | `--target-lang es` |

## 翻译API配置

### 默认API

程序默认使用配置的 DeepLX API：
```
https://deepl3.fileaiwork.online/dptrans?token=ej0ab47388ed86e843de9f499e52e6e664ae1m491cad7bf1.bIrYaAAAAAA=.b9c326068ac3c37ff36b8fea77867db51ddf235150945d7ad43472d68581e6c4pd14&newllm=1
```

### 本地DeepLX服务

如果您有本地DeepLX服务：

```bash
# 使用本地DeepLX
monolith https://example.com -T \
  --translation-api "http://localhost:1188/translate" \
  -o translated.html
```

### API格式要求

翻译API需要支持以下请求格式：

```json
{
  "text": "要翻译的文本",
  "source_lang": "auto",
  "target_lang": "zh"
}
```

响应格式：
```json
{
  "code": 200,
  "data": "翻译后的文本"
}
```

## 实际使用示例

### 示例1: 翻译英文新闻网站

```bash
monolith https://www.bbc.com/news -T --target-lang zh -o bbc_news_zh.html
```

### 示例2: 翻译技术文档

```bash
monolith https://docs.rust-lang.org/book/ -T -I -j --target-lang zh -o rust_book_zh.html
```

### 示例3: 翻译GitHub项目README

```bash
monolith https://github.com/user/project -T --target-lang zh -o project_zh.html
```

### 示例4: 从标准输入翻译

```bash
curl https://example.com | monolith - -T --target-lang zh -o translated.html
```

## 性能和注意事项

### 性能优化

- **并行处理**: 长文档会自动分块并行翻译
- **智能过滤**: 自动跳过不需要翻译的内容（代码、链接、数字等）
- **速率限制**: 内置API请求频率控制，避免超出限制

### 翻译质量

- **保持格式**: 翻译后完全保持原网页的HTML结构和CSS样式
- **代码保护**: 代码块、脚本、样式表等技术内容不会被翻译
- **属性处理**: 图片alt文本、按钮title等用户界面文本会被翻译

### 网络要求

- 需要可访问的翻译API服务
- 建议稳定的网络连接
- API服务的速率限制影响翻译速度

## 故障排除

### 常见问题

1. **翻译功能不可用**
   ```bash
   # 确保编译时启用了翻译功能
   cargo build --features translation
   ```

2. **API连接失败**
   ```bash
   # 检查API地址是否正确
   curl -X POST "https://your-api.com/translate" \
     -H "Content-Type: application/json" \
     -d '{"text":"test","source_lang":"en","target_lang":"zh"}'
   ```

3. **翻译速度慢**
   - 检查网络连接
   - 确认API服务响应时间
   - 考虑使用本地DeepLX服务

4. **部分内容未翻译**
   - 这是正常现象，程序会智能跳过代码、链接等不适合翻译的内容
   - 检查是否为纯数字、符号或技术标记

### 调试模式

```bash
# 非静默模式查看翻译进度
monolith https://example.com -T --target-lang zh -o translated.html

# 查看详细错误信息
RUST_LOG=debug monolith https://example.com -T --target-lang zh -o translated.html
```

## 开发和扩展

### 添加新的翻译API

修改 `src/translation.rs` 中的 `translate_dom_content` 函数，支持新的API格式。

### 自定义翻译逻辑

可以修改 `should_translate` 函数来调整哪些内容需要翻译。

### 集成到其他项目

```rust
use monolith::translation::translate_dom_content_sync;

// 在你的代码中使用翻译功能
let translated_dom = translate_dom_content_sync(dom, "zh", Some("https://api.com/translate"))?;
```

## 更新日志

- ✅ v2.11.0: 集成 translation-lib 翻译功能
- ✅ 支持中文、英文、日文等多种目标语言
- ✅ 智能内容识别和代码块保护
- ✅ 异步并行处理优化
- ✅ 完整的命令行参数支持