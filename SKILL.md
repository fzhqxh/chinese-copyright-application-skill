---
name: chinese-copyright-application
description: "用于生成中国软件著作权申请材料的完整工具包。支持从项目代码、文档等自动提取信息，生成软件著作权登记申请表、源代码文档（前后各30页）、用户手册和设计说明书，并自动转换为PDF文件。适用于微信小程序、Web应用、移动App、桌面应用等各类软件项目。当用户需要申请中国软件著作权时使用此skill。"
---

# 中国软件著作权申请材料生成

Use when the user needs to generate Chinese software copyright registration application materials. Generates four documents: application form, source code document (front/back 30 pages each), user manual, and design specification.

## 工作流程

### 步骤1：收集著作权人信息

询问用户著作权人的名称，确保在每一个生成的文档开头都包含著作权人名称。

### 步骤2：分析项目并提取信息

自动检测项目类型并从配置文件中提取信息。可使用 `scripts/generate_copyright_docs.py` 自动化此步骤：

```bash
python3 scripts/generate_copyright_docs.py <项目路径> [输出目录]
```

检测逻辑：
- 存在 `app.json` → 微信小程序（从 `window.navigationBarTitleText` 取名称）
- 存在 `package.json` → Web/Node.js 项目
- 其他 → 查找 `pom.xml`、`build.gradle`、`Cargo.toml` 等配置文件

### 步骤3：生成申请表

使用 [application-form-template.md](references/application-form-template.md) 模板，填充所有字段。

**关键格式要求：**
- 软件全称必须有辨识度，格式为"xxx软件"
- 版本号保留两位（如"V1.0"、"V1.1"）
- 所有字段参见 [申请要求](references/requirements.md)

**验证：** 检查模板中所有 `{placeholder}` 均已替换，无空字段。

### 步骤4：生成源代码文档

**严格要求：前30页 + 后30页，每页50行，共3000行。代码不足3000行则全部提供。**

提取与分页算法：

1. 收集项目所有代码文件（排除 `node_modules`、`.git`、`dist`、`build`）
2. 按优先级排序：入口文件（`app.js`/`main.js`/`index.js`）→ `utils/` → `pages/` → `components/` → `config/`
3. 统计总代码行数：`find <project> -name "*.js" -o -name "*.ts" -o -name "*.py" | xargs wc -l`
4. 前30页：从优先级最高的文件开始，提取前1500行（30页 × 50行）
5. 后30页：从最后的文件反向提取最后1500行
6. 每个文件添加文件头注释，每50行插入页码标记 `--- 第 N 页 ---`

**验证：** 确认总页数为60页（或代码不足时为实际页数），每页正好50行。

### 步骤5：生成用户手册

使用 [user-manual-template.md](references/user-manual-template.md) 模板，从项目 README 和代码结构中提取功能描述和操作流程。

**验证：** 手册不少于10页，包含功能介绍、操作步骤、注意事项。

### 步骤6：生成设计说明书

使用 [design-doc-template.md](references/design-doc-template.md) 模板，通过分析项目结构、代码逻辑和数据流生成技术文档。

**验证：** 说明书不少于10页，包含需求分析、总体设计、详细设计、数据结构。

### 步骤7：转换为PDF并输出

将所有 Markdown 文档转换为 PDF：

```bash
# 使用 pandoc 转换（推荐，需安装 pandoc + CJK 字体支持）
pandoc 软件著作权登记申请表.md -o 软件著作权登记申请表.pdf --pdf-engine=xelatex -V mainfont="SimSun"

# 或使用 mdpdf / md-to-pdf
npx md-to-pdf 软件著作权登记申请表.md
```

所有输出文件写入 `copyright-application-materials/` 目录：

| 文件 | 格式 |
|------|------|
| 软件著作权登记申请表 | .md + .pdf |
| 源代码文档 | .md + .pdf |
| 用户手册 | .md + .pdf |
| 设计说明书 | .md + .pdf |

### 最终验证清单

- [ ] 所有文档中软件名称、版本号一致
- [ ] 源代码文档页数正确（60页或全部代码页数）
- [ ] 用户手册 ≥ 10页
- [ ] 设计说明书 ≥ 10页
- [ ] 申请表无空字段
- [ ] PDF文件已生成且可正常打开
- [ ] 源代码中无敏感信息（密钥、密码等）

## 项目类型映射

| 项目类型 | 软件分类 | 平台 | 技术特点 |
|----------|----------|------|----------|
| 微信小程序 | 移动应用软件-小程序 | 微信小程序平台 | 微信小程序原生框架 |
| Web应用 | 应用软件-Web应用 | 浏览器 | 前端框架 + 后端技术 |
| 移动App | 移动应用软件-App | iOS/Android | 原生/跨平台框架 |

## 参考文档

- [申请表模板](references/application-form-template.md)
- [用户手册模板](references/user-manual-template.md)
- [设计说明书模板](references/design-doc-template.md)
- [申请要求](references/requirements.md)
