---
name: html-to-pdf
description: 将远程HTML网页转换为本地PDF文件，支持身份验证、自定义请求头、JavaScript渲染和CSS样式。支持单页转换和完整文档转换（自动爬取侧边栏所有子页面）。当用户需要下载网页为PDF、转换HTML到PDF、保存网页内容为文档、或提到完整文档/所有章节时使用。
---

# HTML转PDF转换

将远程HTML网页转换为本地PDF文件，完全控制渲染选项。支持两种模式：

## 转换模式选择

**重要**：首先判断用户需要哪种模式：

1. **单页模式** - 用户只提到"转换这个网页"、"单个页面"
2. **完整文档模式** - 用户提到"所有章节"、"完整文档"、"包含所有页面"、"侧边栏所有链接"

## 快速开始

### 单页转换流程

1. 判断HTML源的复杂度（静态页面 vs JavaScript重度页面）
2. 选择合适的库
3. 如需要，处理身份验证/请求头
4. 配置PDF输出选项
5. 保存到本地文件

### 完整文档转换流程

1. 访问文档首页
2. **真实爬取**侧边栏所有链接（不要猜测URL！）
3. 等待JavaScript加载完成（至少10-15秒）
4. 提取所有文档链接
5. 逐个转换每个页面
6. 合并成单个PDF文件

## 库选择指南

根据页面需求选择：

| 库 | 最适合 | 核心特性 |
|---------|----------|--------------|
| **Playwright** | 现代Web应用、单页应用、JavaScript重度页面 | 完整浏览器自动化、JS执行、截图 |
| **WeasyPrint** | 静态HTML、CSS样式页面 | 纯Python、优秀的CSS支持、无外部依赖 |
| **pdfkit** | 通用场景、混合内容 | wkhtmltopdf封装、良好兼容性 |

**默认推荐**：优先使用Playwright，可靠性和功能完整性最好。

## 基础转换

### 使用 Playwright（推荐）

```python
from playwright.sync_api import sync_playwright

def html转pdf(网址, 输出路径, **选项):
    with sync_playwright() as p:
        浏览器 = p.chromium.launch()
        页面 = 浏览器.new_page()
        页面.goto(网址)
        页面.pdf(path=输出路径, **选项)
        浏览器.close()

# 示例
html转pdf("https://example.com", "输出.pdf")
```

### 使用 WeasyPrint（静态HTML）

```python
from weasyprint import HTML

def html转pdf(网址, 输出路径):
    HTML(网址).write_pdf(输出路径)

# 示例
html转pdf("https://example.com", "输出.pdf")
```

## 高级功能

### 身份验证与请求头

```python
# Playwright带自定义请求头
def html转pdf带认证(网址, 输出路径, 请求头=None):
    with sync_playwright() as p:
        浏览器 = p.chromium.launch()
        上下文 = 浏览器.new_context(extra_http_headers=请求头 or {})
        页面 = 上下文.new_page()
        页面.goto(网址)
        页面.pdf(path=输出路径)
        浏览器.close()

# 带身份验证的示例
请求头 = {
    "Authorization": "Bearer 你的令牌",
    "User-Agent": "自定义User Agent"
}
html转pdf带认证("https://example.com", "输出.pdf", 请求头)
```

### 等待JavaScript渲染

```python
# 等待特定内容加载
def html转pdf带等待(网址, 输出路径, 选择器=None, 超时时间=30000):
    with sync_playwright() as p:
        浏览器 = p.chromium.launch()
        页面 = 浏览器.new_page()
        页面.goto(网址, wait_until="networkidle")
        
        if 选择器:
            页面.wait_for_selector(选择器, timeout=超时时间)
        
        页面.pdf(path=输出路径)
        浏览器.close()

# 等待特定元素
html转pdf带等待("https://example.com", "输出.pdf", 选择器="#content")
```

### PDF格式化选项

```python
# 完全控制PDF输出
def html转pdf格式化(网址, 输出路径):
    with sync_playwright() as p:
        浏览器 = p.chromium.launch()
        页面 = 浏览器.new_page()
        页面.goto(网址)
        页面.pdf(
            path=输出路径,
            format="A4",                    # 纸张大小
            print_background=True,          # 包含背景图形
            margin={                        # 页边距
                "top": "20mm",
                "right": "20mm",
                "bottom": "20mm",
                "left": "20mm"
            },
            display_header_footer=True,     # 显示页眉/页脚
            header_template="<div style='font-size:10px; text-align:center; width:100%;'>我的页眉</div>",
            footer_template="<div style='font-size:10px; text-align:center; width:100%;'>第 <span class='pageNumber'></span> 页，共 <span class='totalPages'></span> 页</div>",
            prefer_css_page_size=False,     # 使用format而非CSS
            landscape=False                 # 纵向方向
        )
        浏览器.close()
```

## 常见工作流

### 单页转换

```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    浏览器 = p.chromium.launch()
    页面 = 浏览器.new_page()
    页面.goto("https://example.com")
    页面.pdf(path="输出.pdf", format="A4", print_background=True)
    浏览器.close()
```

### 批量转换

```python
def 批量html转pdf(网址列表, 输出目录):
    with sync_playwright() as p:
        浏览器 = p.chromium.launch()
        for i, 网址 in enumerate(网址列表):
            页面 = 浏览器.new_page()
            页面.goto(网址)
            输出路径 = f"{输出目录}/页面_{i+1}.pdf"
            页面.pdf(path=输出路径)
            页面.close()
        浏览器.close()

# 转换多个页面
网址列表 = ["https://example.com/page1", "https://example.com/page2"]
批量html转pdf(网址列表, "./pdfs")
```

### 使用自定义CSS转换

```python
def html转pdf带css(网址, 输出路径, 自定义css=None):
    with sync_playwright() as p:
        浏览器 = p.chromium.launch()
        页面 = 浏览器.new_page()
        页面.goto(网址)
        
        # 注入自定义CSS
        if 自定义css:
            页面.add_style_tag(content=自定义css)
        
        页面.pdf(path=输出路径)
        浏览器.close()

# 转换前隐藏元素
自定义css = """
    .advertisement { display: none !important; }
    .navigation { display: none !important; }
"""
html转pdf带css("https://example.com", "输出.pdf", 自定义css)
```

## 错误处理

始终处理常见错误：

```python
def 安全html转pdf(网址, 输出路径):
    try:
        with sync_playwright() as p:
            浏览器 = p.chromium.launch()
            页面 = 浏览器.new_page()
            
            # 设置超时和错误处理
            页面.set_default_timeout(30000)
            响应 = 页面.goto(网址)
            
            if 响应.status != 200:
                raise Exception(f"HTTP {响应.status}: 加载页面失败")
            
            页面.pdf(path=输出路径)
            浏览器.close()
            return True
            
    except Exception as e:
        print(f"转换错误 {网址}: {str(e)}")
        return False
```

## 安装要求

向用户说明所需的包：

**Playwright（推荐，单页和完整文档都需要）**：
```bash
pip install playwright
playwright install chromium
```

**PyPDF2（仅完整文档模式需要，用于合并PDF）**：
```bash
pip install PyPDF2
```

**WeasyPrint（可选，静态HTML单页转换）**：
```bash
pip install weasyprint
```

**pdfkit（可选，备选方案）**：
```bash
pip install pdfkit
# 还需要在系统上安装wkhtmltopdf
```

## 决策树

根据用户需求选择转换模式：

### 第1步：判断转换模式

**用户说了什么？**

- "转换这个网页" / "把这个URL转成PDF" / "单个页面"
  → **使用单页模式**

- "所有章节" / "完整文档" / "包含侧边栏所有页面" / "包含所有子页面"
  → **使用完整文档模式**

### 第2步：选择库（单页模式）

1. **是否需要执行JavaScript？**
   - 是 → 使用Playwright
   - 否 → 继续步骤2

2. **页面是否需要身份验证或自定义请求头？**
   - 是 → 使用Playwright
   - 否 → 继续步骤3

3. **是否为带CSS样式的静态HTML？**
   - 是 → 使用WeasyPrint（更快、更轻）
   - 否 → 使用Playwright（最安全的默认选择）

### 第3步：完整文档模式的关键点

**必须遵守的规则**：

1. ⚠️ **绝对不要猜测URL路径！**
   - ❌ 错误：假设路径是 `/docs/agent/pane`
   - ✅ 正确：从页面上真实提取链接

2. ⚠️ **必须等待JavaScript加载！**
   - ❌ 错误：立即提取（只能找到2-3个链接）
   - ✅ 正确：等待10-15秒后提取（能找到50+个链接）

3. ⚠️ **使用 page.evaluate() 提取链接！**
   - ✅ 在浏览器上下文中运行JavaScript
   - ✅ 能获取动态渲染的内容

4. ⚠️ **需要安装 PyPDF2 来合并！**
   - 如果未安装：`pip install PyPDF2`

## 完整文档模式的详细实现

```python
import time

# 步骤1：真实提取导航链接
def 提取所有文档链接(页面, 首页url):
    """
    关键：真实爬取，不猜测！
    """
    页面.goto(首页url, timeout=60000)
    
    # 重要！等待足够长的时间
    time.sleep(15)
    
    # 使用JavaScript提取所有链接
    链接数据 = 页面.evaluate("""
        () => {
            const links = Array.from(document.querySelectorAll('a'));
            return links.map(a => ({
                text: a.textContent.trim(),
                href: a.href
            })).filter(l => l.text && l.href.includes('/docs/'));
        }
    """)
    
    # 去重
    唯一链接 = {}
    for 项 in 链接数据:
        url = 项['href']
        if url not in 唯一链接:
            唯一链接[url] = 项['text']
    
    return [(标题, url) for url, 标题 in 唯一链接.items()]

# 步骤2：批量转换
def 批量转换并合并(文档列表, 输出文件):
    """
    转换所有页面并合并
    """
    from PyPDF2 import PdfMerger
    import tempfile
    
    临时目录 = tempfile.mkdtemp()
    pdf文件列表 = []
    
    with sync_playwright() as p:
        浏览器 = p.chromium.launch()
        页面 = 浏览器.new_page()
        
        for i, (标题, url) in enumerate(文档列表, 1):
            try:
                页面.goto(url, wait_until="domcontentloaded", timeout=30000)
                time.sleep(1)
                
                # 隐藏导航
                页面.add_style_tag(content="nav, header, .sidebar { display: none !important; }")
                
                pdf路径 = os.path.join(临时目录, f"{i:03d}.pdf")
                页面.pdf(path=pdf路径, format="A4", print_background=True)
                pdf文件列表.append(pdf路径)
                
            except:
                pass
        
        浏览器.close()
    
    # 合并
    merger = PdfMerger()
    for pdf in pdf文件列表:
        merger.append(pdf)
    merger.write(输出文件)
    merger.close()
    
    # 清理
    import shutil
    shutil.rmtree(临时目录)
```

## 常见问题与解决方案

### 单页转换问题

**问题**：PDF为空白或不完整
- **解决方案**：添加 `wait_until="networkidle"` 或等待特定选择器

**问题**：需要身份验证
- **解决方案**：使用 `extra_http_headers` 或带cookies的浏览器上下文

**问题**：背景图形缺失
- **解决方案**：在PDF选项中设置 `print_background=True`

**问题**：页面布局错乱
- **解决方案**：设置合适的 `format`、`margin` 和 `prefer_css_page_size` 选项

**问题**：内容被截断
- **解决方案**：调整边距或使用 `height` 参数进行全页捕获

### 完整文档转换问题

**问题**：只找到2-3个链接，大部分页面缺失
- **原因**：JavaScript还没加载完成就提取了链接
- **解决方案**：在 `page.goto()` 后必须 `time.sleep(15)` 等待足够长时间

**问题**：很多页面返回404
- **原因**：URL路径是猜测的，不是真实爬取的
- **解决方案**：必须使用 `page.evaluate()` 从页面真实提取 `<a>` 标签

**问题**：单页应用（SPA）导航不可见
- **原因**：侧边栏是动态渲染的
- **解决方案**：
  1. 等待时间增加到 15 秒
  2. 尝试滚动页面触发懒加载
  3. 使用 `wait_for_selector()` 等待特定导航元素

**问题**：知乎、微信等网站返回403
- **原因**：反爬虫保护
- **解决方案**：
  1. 添加真实的User-Agent
  2. 使用非无头模式（headless=False）
  3. 添加反检测脚本
  4. 最后手段：建议用户手动打印

**问题**：合并PDF失败
- **原因**：缺少 PyPDF2 库
- **解决方案**：`pip install PyPDF2`

**问题**：日志显示失败但PDF实际生成成功
- **原因**：某些操作（如CSS注入）可能返回None但不影响PDF生成
- **解决方案**：已在新版本修复，现在基于PDF文件是否真实生成来判断成功/失败

## 最佳实践与技巧

### 1. 性能优化

```python
# 对于大量页面转换，复用浏览器实例
def 批量转换优化(url列表, 输出目录):
    with sync_playwright() as p:
        浏览器 = p.chromium.launch()
        页面 = 浏览器.new_page()
        
        for i, url in enumerate(url列表):
            # 复用同一个页面对象，避免频繁创建/销毁
            页面.goto(url, wait_until="domcontentloaded")
            页面.pdf(path=f"{输出目录}/{i:03d}.pdf")
            
        浏览器.close()
```

### 2. 智能等待策略

```python
# 根据页面复杂度动态调整等待时间
def 智能等待(页面对象, 网址):
    页面对象.goto(网址, wait_until="domcontentloaded")
    
    # 检查是否为SPA
    是否spa = 页面对象.evaluate("() => !!window.React || !!window.Vue || !!window.Angular")
    
    if 是否spa:
        print("检测到SPA，等待15秒...")
        time.sleep(15)
    else:
        print("静态页面，等待3秒...")
        time.sleep(3)
```

### 3. 自定义链接过滤

```python
# 使用正则表达式或自定义函数过滤链接
def 高级链接过滤(所有链接, 自定义条件):
    """
    自定义条件示例：
    lambda link: '/docs/' in link['href'] and 'api' not in link['href']
    """
    return [链接 for 链接 in 所有链接 if 自定义条件(链接)]

# 使用示例
文档链接 = 高级链接过滤(
    所有链接,
    lambda l: '/guide/' in l['href'] and len(l['text']) > 3
)
```

### 4. 错误恢复与重试

```python
def 转换带重试(页面对象, 网址, 输出路径, 最大重试=3):
    """失败后自动重试"""
    for 尝试次数 in range(最大重试):
        try:
            响应 = 页面对象.goto(网址, timeout=30000)
            if 响应.status == 200:
                页面对象.pdf(path=输出路径)
                if os.path.exists(输出路径):
                    return True
        except Exception as e:
            if 尝试次数 < 最大重试 - 1:
                print(f"重试 {尝试次数 + 1}/{最大重试}...")
                time.sleep(2 ** 尝试次数)  # 指数退避
            else:
                print(f"失败: {str(e)}")
    return False
```

### 5. 添加书签和目录

```python
def 添加pdf书签(输入pdf, 输出pdf, 书签列表):
    """
    为合并的PDF添加书签
    书签列表格式: [(标题, 页码), ...]
    """
    from PyPDF2 import PdfReader, PdfWriter
    
    reader = PdfReader(输入pdf)
    writer = PdfWriter()
    
    for page in reader.pages:
        writer.add_page(page)
    
    # 添加书签
    for 标题, 页码 in 书签列表:
        writer.add_outline_item(标题, 页码 - 1)  # PyPDF2使用0索引
    
    with open(输出pdf, 'wb') as f:
        writer.write(f)
```

### 6. 保存转换历史

```python
import json
from datetime import datetime

def 保存转换记录(基础网址, 输出路径, 成功数, 总数):
    """记录转换历史，便于追踪和调试"""
    记录 = {
        '时间': datetime.now().isoformat(),
        '源地址': 基础网址,
        '输出文件': 输出路径,
        '成功': 成功数,
        '总数': 总数,
        '成功率': f"{成功数/总数*100:.1f}%"
    }
    
    历史文件 = 'conversion_history.json'
    try:
        with open(历史文件, 'r') as f:
            历史 = json.load(f)
    except:
        历史 = []
    
    历史.append(记录)
    
    with open(历史文件, 'w') as f:
        json.dump(历史, f, indent=2, ensure_ascii=False)
```

### 7. 处理分页文档

```python
def 提取分页链接(页面对象, 基础网址):
    """
    处理带有"下一页"按钮的分页文档
    """
    所有页面 = []
    当前页 = 1
    
    while True:
        print(f"处理第 {当前页} 页...")
        
        # 提取当前页的链接
        链接 = 页面对象.evaluate("""
            () => Array.from(document.querySelectorAll('a'))
                .map(a => ({text: a.textContent.trim(), href: a.href}))
        """)
        所有页面.extend(链接)
        
        # 查找"下一页"按钮
        try:
            下一页 = 页面对象.query_selector('a[aria-label*="next"], a:has-text("下一页")')
            if 下一页:
                下一页.click()
                time.sleep(2)
                当前页 += 1
            else:
                break
        except:
            break
    
    return 所有页面
```

## 关键代码片段

### 提取侧边栏链接的正确方法

```python
# ✅ 正确方法：真实爬取
页面.goto("https://example.com/docs", timeout=60000)
time.sleep(15)  # 关键！等待JavaScript渲染

链接 = 页面.evaluate("""
    () => {
        const links = Array.from(document.querySelectorAll('a'));
        return links
            .filter(a => a.href.includes('/docs/'))
            .map(a => ({text: a.textContent.trim(), href: a.href}));
    }
""")

# ❌ 错误方法：猜测URL
文档列表 = [
    ("概览", "https://example.com/docs/overview"),  # 可能是错的！
    ("安装", "https://example.com/docs/install"),   # 可能是错的！
]
```

### 处理反爬虫保护

```python
# 添加反检测
页面.add_init_script("""
    Object.defineProperty(navigator, 'webdriver', {
        get: () => undefined
    });
""")

# 使用真实浏览器设置
上下文 = 浏览器.new_context(
    viewport={'width': 1920, 'height': 1080},
    user_agent='Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36',
    locale='zh-CN'
)
```

## 输出验证

转换后验证PDF：

```python
import os

def 验证pdf已创建(输出路径, 最小大小kb=10):
    if not os.path.exists(输出路径):
        print(f"❌ PDF未创建: {输出路径}")
        return False
    
    大小kb = os.path.getsize(输出路径) / 1024
    if 大小kb < 最小大小kb:
        print(f"⚠️  PDF文件过小可能有问题: {大小kb:.1f} KB")
        return False
    
    print(f"✅ PDF创建成功: {大小kb:.1f} KB")
    return True
```

## 实际使用示例

### 示例1：单个网页

**用户说**："转换 https://example.com 为PDF"

**代码**：
```python
from playwright.sync_api import sync_playwright

with sync_playwright() as p:
    浏览器 = p.chromium.launch()
    页面 = 浏览器.new_page()
    页面.goto("https://example.com", wait_until="networkidle")
    页面.pdf(path="输出.pdf", format="A4", print_background=True)
    浏览器.close()
```

### 示例2：完整文档（关键示例！）

**用户说**："转换 https://cursor.com/cn/docs 完整文档，包含所有侧边栏章节"

**代码**：
```python
from playwright.sync_api import sync_playwright
from PyPDF2 import PdfMerger
import os, time

# 第1步：真实爬取所有链接（不要猜！）
with sync_playwright() as p:
    浏览器 = p.chromium.launch()
    页面 = 浏览器.new_page()
    
    # 访问首页
    页面.goto("https://cursor.com/cn/docs", timeout=60000)
    
    # 关键！等待15秒让JavaScript加载
    time.sleep(15)
    
    # 真实提取链接
    链接 = 页面.evaluate("""
        () => {
            return Array.from(document.querySelectorAll('a'))
                .map(a => ({text: a.textContent.trim(), href: a.href}))
                .filter(l => l.text && l.href.includes('/docs/'));
        }
    """)
    
    # 去重
    文档列表 = []
    已见 = set()
    for 项 in 链接:
        if 项['href'] not in 已见:
            已见.add(项['href'])
            文档列表.append((项['text'], 项['href']))
    
    print(f"找到 {len(文档列表)} 个文档页面")
    
    # 第2步：批量转换
    临时目录 = "/tmp/pdfs"
    os.makedirs(临时目录, exist_ok=True)
    pdf列表 = []
    
    for i, (标题, url) in enumerate(文档列表, 1):
        try:
            页面.goto(url, wait_until="domcontentloaded", timeout=30000)
            time.sleep(1)
            页面.add_style_tag(content="nav, header { display: none !important; }")
            
            pdf路径 = f"{临时目录}/{i:03d}.pdf"
            页面.pdf(path=pdf路径, format="A4", print_background=True)
            pdf列表.append(pdf路径)
            print(f"[{i}/{len(文档列表)}] ✅ {标题}")
        except:
            print(f"[{i}/{len(文档列表)}] ❌ {标题}")
    
    浏览器.close()
    
    # 第3步：合并
    merger = PdfMerger()
    for pdf in pdf列表:
        merger.append(pdf)
    merger.write("完整文档.pdf")
    merger.close()
    
    print(f"✅ 完成！生成了完整文档.pdf")
```

## 完整工作流示例

### 模式1：单页转换

```python
from playwright.sync_api import sync_playwright
import os

def 转换单个网页(网址, 输出路径, **选项):
    """
    转换单个网页为PDF
    
    参数：
        网址: 要转换的远程URL
        输出路径: 保存PDF的本地路径
        **选项: 额外选项（请求头、等待选择器、自定义CSS、PDF选项）
    """
    请求头 = 选项.get('请求头', {})
    等待选择器 = 选项.get('等待选择器')
    自定义css = 选项.get('自定义css')
    pdf选项 = 选项.get('pdf选项', {})
    
    默认pdf选项 = {
        'format': 'A4',
        'print_background': True,
        'margin': {'top': '20mm', 'right': '20mm', 'bottom': '20mm', 'left': '20mm'}
    }
    默认pdf选项.update(pdf选项)
    
    try:
        with sync_playwright() as p:
            浏览器 = p.chromium.launch()
            上下文 = 浏览器.new_context(extra_http_headers=请求头) if 请求头 else 浏览器
            页面 = 上下文.new_page() if 请求头 else 浏览器.new_page()
            
            响应 = 页面.goto(网址, wait_until="networkidle")
            
            if 响应.status != 200:
                raise Exception(f"HTTP {响应.status}")
            
            if 等待选择器:
                页面.wait_for_selector(等待选择器, timeout=30000)
            
            if 自定义css:
                页面.add_style_tag(content=自定义css)
            
            页面.pdf(path=输出路径, **默认pdf选项)
            浏览器.close()
            
            if os.path.exists(输出路径):
                大小kb = os.path.getsize(输出路径) / 1024
                print(f"✅ PDF已创建: {输出路径} ({大小kb:.1f} KB)")
                return True
            else:
                print(f"❌ PDF创建失败")
                return False
                
    except Exception as e:
        print(f"❌ 错误: {str(e)}")
        return False
```

### 模式2：完整文档转换（自动爬取侧边栏）

**关键要点**：
- ❌ **不要猜测URL路径**
- ✅ **必须真实爬取页面上的链接**
- ✅ **等待JavaScript加载完成**（SPA需要时间）

```python
from playwright.sync_api import sync_playwright
import os
import time

def 提取真实导航链接(页面对象, 基础网址, 链接过滤关键词=None):
    """
    从页面上真实提取所有文档链接
    
    关键步骤：
    1. 访问文档首页
    2. 等待JavaScript加载（10-15秒）
    3. 提取所有<a>标签
    4. 过滤出文档链接
    
    参数：
        页面对象: Playwright页面对象
        基础网址: 文档首页URL
        链接过滤关键词: 可选的URL过滤关键词列表，如 ['/docs/', '/api/']
    """
    print("正在访问文档首页...")
    页面对象.goto(基础网址, timeout=60000)
    
    print("等待JavaScript加载完成（15秒）...")
    time.sleep(15)  # 重要！SPA需要时间渲染
    
    print("提取所有文档链接...")
    所有链接 = 页面对象.evaluate("""
        () => {
            // 获取页面上所有链接
            const links = Array.from(document.querySelectorAll('a'));
            return links.map(a => ({
                text: a.textContent.trim(),
                href: a.href,
                className: a.className
            })).filter(l => l.text && l.href);
        }
    """)
    
    # 自动推断过滤关键词（如果未提供）
    if 链接过滤关键词 is None:
        from urllib.parse import urlparse
        基础路径 = urlparse(基础网址).path
        链接过滤关键词 = [基础路径] if 基础路径 else ['/docs/', '/api/', '/guide/']
    
    # 过滤文档链接
    文档链接 = []
    已见url = set()
    
    # 常见的要跳过的标题（通用列表）
    跳过标题 = ['Logo', 'Docs', 'API', 'Guide', 'Documentation', 
               'Skip to', 'Menu', 'Toggle', 'Search', 'GitHub',
               '文档', '指南', '菜单', '搜索', '跳转']
    
    for 链接 in 所有链接:
        url = 链接['href']
        标题 = 链接['text']
        
        # 检查URL是否匹配任一过滤关键词
        url匹配 = any(关键词 in url for 关键词 in 链接过滤关键词)
        
        if url匹配 and url not in 已见url:
            # 跳过无意义的标题
            if 标题 and not any(跳过词 in 标题 for 跳过词 in 跳过标题):
                已见url.add(url)
                文档链接.append((标题, url))
    
    print(f"✅ 找到 {len(文档链接)} 个文档页面")
    
    # 如果找到的链接很少，给出提示
    if len(文档链接) < 5:
        print(f"⚠️  找到的链接较少，可能需要：")
        print(f"   1. 增加等待时间（当前15秒）")
        print(f"   2. 检查过滤关键词：{链接过滤关键词}")
        print(f"   3. 滚动页面触发懒加载")
    
    return 文档链接

def 转换单个页面(页面对象, 网址, 标题, 临时目录, 序号, 总数):
    """
    转换单个页面为PDF
    
    关键改进：基于PDF文件是否生成来判断成功/失败，而非异常
    """
    输出 = None
    try:
        print(f"  [{序号}/{总数}] {标题[:40]}", end=" ", flush=True)
        响应 = 页面对象.goto(网址, wait_until="domcontentloaded", timeout=30000)
        
        if 响应.status != 200:
            print(f"❌ HTTP {响应.status}")
            return None
        
        time.sleep(1)
        
        # 注入优化CSS（隐藏导航等）
        # 注意：add_style_tag() 可能返回 None，但不影响PDF生成
        try:
            页面对象.add_style_tag(content="""
                nav, header, .sidebar, button, footer { display: none !important; }
                body { background: white !important; font-size: 10pt !important; }
                h1 { font-size: 15pt !important; page-break-after: avoid; }
                h2 { font-size: 12pt !important; page-break-after: avoid; }
                pre, code { background: #f5f5f5 !important; border: 1px solid #ddd !important; 
                            font-size: 8pt !important; page-break-inside: avoid; }
                table { border-collapse: collapse !important; font-size: 9pt !important; }
                th, td { border: 1px solid #ddd !important; padding: 4px !important; }
            """)
        except:
            pass  # CSS注入失败不影响PDF生成
        
        安全名 = "".join(c if c.isalnum() or c in (' ', '-', '_') else '_' for c in 标题)
        输出 = os.path.join(临时目录, f"{序号:03d}_{安全名[:30]}.pdf")
        
        # 生成PDF
        页面对象.pdf(path=输出, format="A4", print_background=True,
                    margin={"top": "10mm", "right": "8mm", "bottom": "10mm", "left": "8mm"})
        
        # 关键：验证PDF文件是否真实生成
        if os.path.exists(输出) and os.path.getsize(输出) > 1024:  # 至少1KB
            print("✅")
            return 输出
        else:
            print("❌ 文件未生成")
            return None
        
    except Exception as e:
        print(f"❌ {str(e)[:30]}")
        return None

def 合并pdf文件(pdf列表, 输出路径):
    """合并多个PDF为单个文件"""
    try:
        from PyPDF2 import PdfMerger
        print(f"\n正在合并 {len(pdf列表)} 个PDF文件...")
        
        merger = PdfMerger()
        for pdf in pdf列表:
            if pdf and os.path.exists(pdf):
                merger.append(pdf)
        
        merger.write(输出路径)
        merger.close()
        print("✅ 合并完成")
        return True
    except ImportError:
        print("❌ 需要安装 PyPDF2: pip install PyPDF2")
        return False
    except Exception as e:
        print(f"❌ 合并失败: {str(e)}")
        return False

def 转换完整文档(基础网址, 输出路径, 链接过滤关键词=None):
    """
    转换完整文档（自动爬取所有章节）
    
    参数：
        基础网址: 文档首页URL（如 https://example.com/docs）
        输出路径: 输出PDF的完整路径
        链接过滤关键词: 可选的URL过滤关键词列表，如 ['/docs/', '/api/']
    
    返回：
        True 表示成功，False 表示失败
    """
    临时目录 = os.path.join(os.path.dirname(输出路径), "temp_pdfs")
    os.makedirs(临时目录, exist_ok=True)
    
    print("=" * 70)
    print("完整文档转换")
    print("=" * 70)
    print(f"目标: {基础网址}")
    print(f"输出: {输出路径}")
    print("=" * 70)
    print()
    
    try:
        with sync_playwright() as p:
            print("启动浏览器...")
            浏览器 = p.chromium.launch()
            页面 = 浏览器.new_page(viewport={'width': 1920, 'height': 1080})
            
            # 第1步：真实爬取所有文档链接
            文档列表 = 提取真实导航链接(页面, 基础网址, 链接过滤关键词)
            
            if not 文档列表:
                print("❌ 未找到任何文档链接")
                浏览器.close()
                return False
            
            总数 = len(文档列表)
            print(f"\n开始转换 {总数} 个页面...\n")
            
            # 显示前几个页面预览
            if 总数 > 0:
                print("前10个页面预览：")
                for i, (标题, _) in enumerate(文档列表[:10], 1):
                    print(f"  {i}. {标题}")
                if 总数 > 10:
                    print(f"  ... 还有 {总数 - 10} 个\n")
            
            # 第2步：逐个转换
            pdf列表 = []
            成功 = 0
            失败 = 0
            
            for i, (标题, url) in enumerate(文档列表, 1):
                pdf = 转换单个页面(页面, url, 标题, 临时目录, i, 总数)
                if pdf:
                    pdf列表.append(pdf)
                    成功 += 1
                else:
                    失败 += 1
                time.sleep(0.6)  # 避免请求过快
            
            浏览器.close()
            
            print(f"\n{'='*70}")
            print(f"转换完成：✅ {成功} 个成功，❌ {失败} 个失败（共 {总数} 个）")
            print(f"{'='*70}")
            
            # 第3步：合并PDF
            if pdf列表:
                if 合并pdf文件(pdf列表, 输出路径):
                    大小mb = os.path.getsize(输出路径) / (1024 * 1024)
                    
                    # 统计PDF总页数
                    try:
                        from PyPDF2 import PdfReader
                        reader = PdfReader(输出路径)
                        总页数 = len(reader.pages)
                        页数信息 = f"   总页数: {总页数} 页"
                    except:
                        页数信息 = ""
                    
                    print(f"\n🎉 完整文档已生成！")
                    print(f"   文件位置: {输出路径}")
                    print(f"   文件大小: {大小mb:.2f} MB")
                    print(f"   包含章节: {成功} 个")
                    if 页数信息:
                        print(页数信息)
                    
                    # 清理临时文件
                    print(f"\n清理临时文件...")
                    import shutil
                    shutil.rmtree(临时目录)
                    print(f"✅ 临时文件已清理")
                    
                    return True
            else:
                print("\n❌ 没有成功转换任何页面")
                return False
            
            return False
            
    except Exception as e:
        print(f"\n❌ 错误: {str(e)}")
        import traceback
        traceback.print_exc()
        return False

# 使用示例
# 单页模式
转换单个网页("https://example.com", "输出.pdf")

# 完整文档模式
转换完整文档("https://example.com/docs", "完整文档.pdf")

# 完整文档模式（自定义过滤）
转换完整文档("https://example.com/docs", "完整文档.pdf", 链接过滤关键词=['/docs/', '/api/'])
```

---

## 快速参考表

| 场景 | 推荐方法 | 关键参数 |
|------|---------|---------|
| 单个静态页面 | `转换单个网页()` | `wait_until="networkidle"` |
| 单个动态页面 | `转换单个网页()` + 等待选择器 | `等待选择器="#content"` |
| 完整文档站点 | `转换完整文档()` | `链接过滤关键词=['/docs/']` |
| 需要身份验证 | `转换单个网页()` + 请求头 | `请求头={'Authorization': '...'}` |
| 自定义样式 | `转换单个网页()` + CSS | `自定义css="nav { display: none }"` |
| 反爬虫网站 | 添加User-Agent + 非无头模式 | `headless=False` |
| 大量页面 | 复用浏览器实例 | 见"性能优化"章节 |
| 添加书签 | `添加pdf书签()` | 见"最佳实践"章节 |

## 常用PDF选项速查

```python
pdf选项 = {
    'format': 'A4',                      # 纸张: A4, Letter, Legal, A3
    'print_background': True,            # 打印背景
    'landscape': False,                  # 横向/纵向
    'margin': {
        'top': '20mm',
        'right': '20mm', 
        'bottom': '20mm',
        'left': '20mm'
    },
    'display_header_footer': True,       # 显示页眉页脚
    'prefer_css_page_size': False,       # 使用CSS定义的尺寸
    'scale': 1.0                         # 缩放比例 (0.1-2.0)
}
```

## 检查清单

转换前检查：
- [ ] 确认Python已安装Playwright（`pip install playwright`）
- [ ] 确认Chromium已安装（`playwright install chromium`）
- [ ] 完整文档模式需要PyPDF2（`pip install PyPDF2`）
- [ ] 输出目录存在且有写权限
- [ ] 网络连接正常

转换完整文档时：
- [ ] 等待JavaScript加载（至少15秒）
- [ ] 使用`page.evaluate()`提取真实链接
- [ ] 不要猜测URL路径
- [ ] 设置合适的链接过滤关键词
- [ ] 检查PDF文件大小和页数是否合理

调试问题时：
- [ ] 查看终端输出的错误信息
- [ ] 检查找到的链接数量是否正常
- [ ] 验证单个页面是否能访问
- [ ] 查看临时PDF文件是否生成
- [ ] 测试单页转换是否成功

---

**记住**：完整文档转换的关键是**真实爬取**而非**猜测URL**！
