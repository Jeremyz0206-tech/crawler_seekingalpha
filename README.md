
# 🕷️ 使用 Playwright 爬取 SeekingAlpha 新闻标题（macOS 桌面保存版）

## 📌 教程目录

1. [环境安装](#环境安装)
2. [爬取目标](#爬取目标)
3. [完整脚本（保存到桌面）](#完整脚本保存到桌面)
4. [输出效果](#输出效果)
5. [可选增强](#可选增强)

---

## ✅ 环境安装

```bash
pip install playwright nest_asyncio pandas tqdm
playwright install
```

---

## 🎯 爬取目标

从 SeekingAlpha 网站爬取如下股票的新闻标题和链接：

- `https://seekingalpha.com/symbol/AAPL/news`
- `https://seekingalpha.com/symbol/TSLA/news`
- `https://seekingalpha.com/symbol/NVDA/news`

---

## 💻 完整脚本（保存到桌面）

```python
import pandas as pd
from tqdm.asyncio import tqdm
import asyncio
from playwright.async_api import async_playwright
import nest_asyncio
import os

nest_asyncio.apply()

desktop_path = os.path.join(os.path.expanduser("~"), "Desktop")
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

# 执行任务
await asyncio.gather(*(scrape_symbol(symbol) for symbol in symbols))
```

---

## 📁 输出效果

- 在桌面生成：
  - `aapl_news_playwright.csv`
  - `tsla_news_playwright.csv`
  - `nvda_news_playwright.csv`
- 每个文件包括新闻标题和链接

---


