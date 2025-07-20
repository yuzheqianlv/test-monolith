# Monolith with Translation Support

基于 monolith 的增强版本，集成了智能翻译功能，可以将网页内容下载并翻译成指定语言的单文件 HTML。

## 项目结构

```
test-monolith/
├── monolith/              # 主项目目录（基于原版 monolith）
│   ├── src/               # 源代码
│   │   ├── translation.rs # 新增：翻译功能模块
│   │   ├── main.rs        # 主程序入口
│   │   ├── core.rs        # 核心逻辑
│   │   └── ...            # 其他模块
│   ├── translation-config.toml # 翻译配置文件
│   └── target/            # 编译输出
├── translation-lib/       # 翻译依赖库（只读参考）
│   ├── src/               # 翻译库源码
│   │   ├── config.rs      # 配置管理
│   │   ├── translator.rs  # 翻译服务
│   │   └── types.rs       # 类型定义
│   └── Cargo.toml         # 依赖配置
└── README.md              # 本文件
```

## 主要功能

### 基础功能（继承自 monolith）
- 将网页及其所有资源（CSS、JavaScript、图片等）打包成单个 HTML 文件
- 支持本地文件和远程 URL
- 可自定义输出格式和压缩选项
- 支持代理和认证

### 新增翻译功能
- **智能批量翻译**：自动将网页文本内容翻译成指定语言
- **配置文件支持**：灵活的 TOML 配置文件管理
- **高性能处理**：优化的批处理和并发翻译
- **智能文本过滤**：自动跳过代码块、URL、邮箱等不需要翻译的内容
- **多API支持**：支持 DeepLX 和其他兼容的翻译 API

## 安装和编译

### 依赖要求
- Rust 1.70+
- Cargo

### 编译步骤

```bash
# 克隆项目
git clone <repository-url>
cd test-monolith/monolith

# 编译（启用翻译功能）
cargo build --release --features "cli,translation"

# 可执行文件位置
./target/release/monolith
```

## 使用方法

### 基础用法

```bash
# 下载网页到单文件（不翻译）
./target/release/monolith "https://example.com" -o example.html

# 下载并翻译成中文
./target/release/monolith "https://example.com" -T --target-lang zh -o example_zh.html

# 使用自定义翻译API
./target/release/monolith "https://example.com" -T --target-lang zh --translation-api "http://localhost:1188/translate" -o example_zh.html
```

### 配置文件管理

```bash
# 生成示例配置文件
./target/release/monolith --generate-config

# 编辑配置文件
nano translation-config.toml
```

### 翻译配置文件示例

```toml
[translation]
# 是否启用翻译功能
enabled = true

# 源语言代码，"auto" 表示自动检测
source_lang = "auto"

# 目标语言代码
target_lang = "zh"

# DeepLX API 地址
deeplx_api_url = "http://localhost:1188/translate"

# 每秒最大请求数（避免 API 限制）
max_requests_per_second = 5.0

# 单次翻译的最大文本长度（字符数）
max_text_length = 50000

# 单次请求的最大段落数
max_paragraphs_per_request = 100
```

## 命令行参数

### 翻译相关参数

- `-T, --translate`: 启用翻译功能
- `--target-lang <LANG>`: 目标语言代码（如：zh, en, ja, fr）
- `--translation-api <URL>`: 翻译 API 地址
- `--generate-config`: 生成示例配置文件

### 基础参数（继承自 monolith）

- `-o, --output <FILE>`: 输出文件路径
- `-c, --no-css`: 移除 CSS
- `-j, --no-js`: 移除 JavaScript
- `-i, --no-images`: 移除图片
- `-s, --silent`: 静默模式
- `--timeout <SECONDS>`: 网络超时时间

完整参数列表请运行：`./target/release/monolith --help`

## 实际使用示例

### 示例 1：翻译技术文档

```bash
# 下载并翻译 Kagi 搜索帮助页面
./target/release/monolith "https://help.kagi.com/kagi/" -T --target-lang zh -o kagi_help_zh.html
```

### 示例 2：批量处理

```bash
# 创建脚本批量翻译多个页面
#!/bin/bash
urls=(
    "https://help.kagi.com/kagi/why-kagi/kagi-vs-competition.html"
    "https://help.kagi.com/kagi/getting-started/"
    "https://help.kagi.com/kagi/features/"
)

for url in "${urls[@]}"; do
    filename=$(basename "$url").html
    ./target/release/monolith "$url" -T --target-lang zh -o "translated_$filename"
    echo "Translated: $url -> translated_$filename"
done
```

### 示例 3：配置不同语言

```bash
# 翻译成日文
./target/release/monolith "https://example.com" -T --target-lang ja -o example_ja.html

# 翻译成法文
./target/release/monolith "https://example.com" -T --target-lang fr -o example_fr.html

# 翻译成德文
./target/release/monolith "https://example.com" -T --target-lang de -o example_de.html
```

## 性能优化特性

### 智能批处理
- 自动将多个小文本合并成批次进行翻译
- 按文本长度智能分组，减少 API 调用次数
- 支持并发翻译处理，提高整体速度

### 文本过滤优化
- 自动识别并跳过代码块、URL、邮箱地址
- 检测已翻译的中文内容，避免重复翻译
- 智能处理 HTML 属性文本（title、alt、placeholder 等）

### 错误处理和恢复
- 内置重试机制，自动处理网络异常
- 批量翻译失败时自动回退到单独翻译
- 详细的错误日志和进度提示

## 配置文件位置优先级

系统会按以下顺序查找配置文件：

1. `translation-config.toml`（当前目录）
2. `config.toml`（当前目录）
3. `.translation-config.toml`（当前目录）
4. `~/.config/monolith/translation.toml`（用户配置目录）
5. `/etc/monolith/translation.toml`（系统配置目录）

## 支持的翻译 API

### DeepLX（推荐）
- 本地部署：`http://localhost:1188/translate`
- 远程服务：需要有效的 token

### 其他兼容 API
项目设计为可扩展，支持任何兼容 DeepL API 格式的翻译服务。

## 开发和贡献

### 项目架构
- `monolith/src/translation.rs`: 翻译功能的核心实现
- `translation-lib/`: 独立的翻译库，可复用于其他项目
- 配置系统基于 TOML，支持热加载和命令行覆盖

### 扩展开发
- 翻译功能模块化设计，易于添加新的翻译服务
- 支持自定义文本过滤规则
- 完整的错误处理和日志系统

## 许可证

本项目基于原版 monolith 项目，继承其开源许可证。

## 技术支持

如遇到问题或需要功能建议，请：
1. 检查配置文件是否正确
2. 确认翻译 API 服务可用
3. 查看详细的错误日志输出

---

**注意**：翻译功能需要配置有效的翻译 API 服务。建议本地部署 DeepLX 以获得最佳性能和隐私保护。
