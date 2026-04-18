# 🎯 HTML转PDF - 通用Agent Skill

<div align="center">

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Compatible](https://img.shields.io/badge/compatible-Cursor%20%7C%20Claude%20%7C%20Copilot%20%7C%2030%2B%20tools-blue)
![Language](https://img.shields.io/badge/language-Python-yellow.svg)

**将远程HTML网页转换为本地PDF文件**

支持单页转换和完整文档转换（自动爬取侧边栏）

</div>

---

## ✨ 核心特性

- 🚀 **智能爬取** - 自动识别SPA，真实提取文档链接（非猜测）
- 📄 **完整文档模式** - 爬取整个文档站点并合并为单个PDF
- 🎨 **完全控制** - 自定义页面、边距、样式、认证
- ✅ **生产验证** - 已成功转换多个大型文档站点

## 🌐 兼容性

遵循 [Agent Skills 开放标准](https://agentskills.io/)，支持：

✅ **Cursor** | ✅ **Claude Code** | ✅ **Claude.ai** | ✅ **GitHub Copilot** | ✅ **VS Code**

以及所有支持 Agent Skills 的工具（30+）

## 📦 安装

```bash
# 克隆到全局技能目录（推荐）
git clone https://github.com/jamesxjz/html-to-pdf-agent-skill.git ~/.cursor/skills/html-to-pdf
```

**依赖：**
```bash
pip install playwright
playwright install chromium
pip install PyPDF2  # 完整文档模式需要
```

## 🚀 使用方法

### 单页转换
```
/html-to-pdf https://example.com
```

### 完整文档转换（所有章节）
```
/html-to-pdf https://cursor.com/cn/docs 把这个页面所有文档转为pdf
```

## 🎯 实战案例

- ✅ Cursor API文档（96页 → 217页PDF）
- ✅ Cursor中文文档（38章节 → 192页）
- ✅ DeepSeek API文档（18章节 → 50页）
- ✅ AgentSkills文档（11章节 → 65页）

## 🔥 核心优势

### 真实爬取 vs 猜测URL

❌ **其他工具**：
```python
urls = ["https://example.com/docs/overview"]  # 经常404
```

✅ **本技能**：
```python
页面.goto("https://example.com/docs")
time.sleep(15)  # 等待SPA加载
链接 = 页面.evaluate("() => Array.from(document.querySelectorAll('a'))")
```

## 📖 文档

详见 [SKILL.md](SKILL.md) 完整指南

## 🤝 贡献

欢迎 Issue 和 Pull Request！

## 📄 License

MIT License - 详见 [LICENSE](LICENSE)

---

<div align="center">

⭐ **如果觉得有用，请给个 Star！** ⭐

Made with ❤️ for the AI coding community

</div>
