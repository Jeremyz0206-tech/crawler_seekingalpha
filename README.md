
# 🕷️ 使用 Playwright 爬取 SeekingAlpha 新闻标题（macOS 教程）

## 📌 目录

1. [安装环境](#1-安装环境)
2. [示例目标](#2-示例目标)
3. [完整脚本（适配 Jupyter 和 .py 文件）](#3-完整脚本适配-jupyter-和-py-文件)
4. [运行效果](#4-运行效果)
5. [可选进阶：滚动加载与摘要抓取](#5-可选进阶滚动加载与摘要抓取)

---

## ✅ 1. 安装环境

只需一次性执行以下命令：

```bash
pip install playwright nest_asyncio pandas tqdm
playwright install
```

---

## 📈 2. 示例目标

爬取以下网址中最新的新闻标题和链接（可换股票）：

```
https://seekingalpha.com/symbol/AAPL/news
https://seekingalpha.com/symbol/TSLA/news
```

---

## 💻 3. 完整脚本（适配 Jupyter 和 .py 文件）

```import pandas as pd
from tqdm.asyncio import tqdm
import asyncio
from playwright.async_api import async_playwright
import nest_asyncio
import os

# Jupyter Notebook 补丁：解决 asyncio loop 冲突
nest_asyncio.apply()

# 获取用户桌面路径
desktop_path = os.path.join(os.path.expanduser("~"), "Desktop")

# 股票代码列表（可自定义）
symbols = ["AAPL", "TSLA", "NVDA"]

async def scrape_symbol(symbol):
    print(f"\n🔍 Scraping {symbol}...")

    async with async_playwright() as p:
        browser = await p.chromium.launch(headless=False)
        context = await browser.new_context()
        page = await context.new_page()

        url = f"https://seekingalpha.com/symbol/{symbol}/news"
        await page.goto(url, timeout=60000)

        try:
            await page.wait_for_selector('a[data-test-id="post-list-item-title"]', timeout=15000)
        except:
            print(f"❌ Failed to load articles for {symbol}")
            await browser.close()
            return

        article_elements = await page.query_selector_all('a[data-test-id="post-list-item-title"]')
        data = []
        for tag in article_elements:
            title = await tag.inner_text()
            href = await tag.get_attribute("href")
            full_url = "https://seekingalpha.com" + href
            data.append({"title": title.strip(), "url": full_url})

        await browser.close()

        if data:
            df = pd.DataFrame(data)
            filename = os.path.join(desktop_path, f"{symbol.lower()}_news_playwright.csv")
            df.to_csv(filename, index=False, encoding="utf-8-sig")
            print(f"✅ Saved {len(df)} articles to {filename}")
        else:
            print(f"⚠️ No news found for {symbol}")

# 启动任务
await asyncio.gather(*(scrape_symbol(symbol) for symbol in symbols))

```

---

## 📂 4. 运行效果

- 输出文件：
  - `aapl_news_playwright.csv`
  - `tsla_news_playwright.csv`
- 每个 CSV 含标题和链接：

| title                              | url                                                |
|------------------------------------|-----------------------------------------------------|
| Apple stock rallies on Q2 earnings | https://seekingalpha.com/news/123456-aapl-earnings |

---

## 🚀 5. 可选进阶：滚动加载与摘要抓取

你可以扩展此脚本实现：

- 自动向下滚动加载更多新闻
- 抓取摘要、发布时间、作者
- 合并多股票结果为一个 DataFrame 输出 Excel

---

🧠 本项目适合用于新闻爬虫练习、投资研究、或构建 RSS 替代方案。
