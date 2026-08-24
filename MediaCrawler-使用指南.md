# MediaCrawler 小红书抓取上手指南（最简版）

> 场景：你是小红书运营/创作者，想用 [MediaCrawler](https://github.com/NanmiCoder/MediaCrawler) 抓竞品笔记、关键词选题、评论区反馈。
>
> 说明：MediaCrawler 是和 TrendRadar **完全独立**的项目，不需要装进本仓库，跟 TrendRadar 的代码没有任何关系。本指南只是把最常用的一套配置和命令整理好，方便你直接抄作业。
>
> ⚠️ 抓取用的是你本人（建议用小号）登录小红书 App/网页后保留的登录态，属于灰色操作，**只做学习/自用分析，不要大规模高频抓，也不要拿数据做商业转卖**，账号有被限流/封禁的风险。

## 0. 准备工作

- 一台能装 Chrome 浏览器的电脑（Windows / macOS / Linux 均可）
- Chrome 浏览器（版本 ≥ 144），装最新版
- 一个小红书小号，手机上装好 App，方便扫码登录
- [uv](https://docs.astral.sh/uv/getting-started/installation)（Python 包管理工具，比 pip 快很多）
- [Node.js](https://nodejs.org/en/download/)（版本 ≥ 16）

## 1. 克隆并安装依赖

```shell
git clone https://github.com/NanmiCoder/MediaCrawler.git
cd MediaCrawler

# 一条命令装好 Python 版本 + 依赖
uv sync
```

## 2. 打开 Chrome 远程调试（用于登录态复用，防止小号被风控）

1. 打开 Chrome，地址栏输入：`chrome://inspect/#remote-debugging`
2. 勾选 **"Allow remote debugging for this browser instance"**
3. 页面显示 `Server running at: 127.0.0.1:9222` 说明已就绪，保持这个 Chrome 窗口开着，不要关

MediaCrawler 默认就是连接你这个已打开的 Chrome（CDP 模式），复用真实浏览器的 Cookie/指纹，比重新弹一个自动化浏览器更不容易被识别成机器人。

## 3. 改配置：只改关键词，其余保持默认即可

打开 `config/xhs_config.py`，把关键词换成你自己的选题方向（英文逗号分隔）：

```python
KEYWORDS = "你的关键词1,你的关键词2"
```

打开 `config/base_config.py`，确认/调整下面几项（默认值已经比较保守，新手建议直接用默认，不用改）：

```python
PLATFORM = "xhs"
LOGIN_TYPE = "qrcode"          # 扫码登录，最简单
CRAWLER_TYPE = "search"        # search=关键词搜索，也可以改成 creator 抓指定账号主页

CRAWLER_MAX_NOTES_COUNT = 15   # 单次最多抓多少篇笔记，新手建议先设小一点，比如 10~20
ENABLE_GET_COMMENTS = True     # 抓评论，做用户反馈/评论情绪分析很有用
CRAWLER_MAX_SLEEP_SEC = 2      # 请求间隔秒数，数字越大越安全，不建议调小

SAVE_DATA_OPTION = "jsonl"     # 想直接拿 Excel 打开分析，改成 "excel" 更方便
```

如果想抓指定竞品账号主页而不是关键词搜索，把 `CRAWLER_TYPE` 改成 `"creator"`，然后在 `config/xhs_config.py` 的 `XHS_CREATOR_ID_LIST` 里填对方主页链接（要带 `xsec_token` 参数，直接从浏览器地址栏复制完整链接即可）。

## 4. 运行

```shell
uv run main.py --platform xhs --lt qrcode --type search
```

第一次运行会弹出小红书二维码，**用小号扫码确认登录**（60 秒内操作），之后登录态会缓存在本地 `browser_data/` 目录下，之后再跑就不用重复扫码了，除非登录态过期。

## 5. 看结果

- 默认存在项目根目录下的 `data/` 文件夹里
- `SAVE_DATA_OPTION = "excel"` 的话直接双击打开 Excel 表格看
- 里面包含笔记标题、正文、点赞收藏数、评论内容等字段，可以直接拿去做选题分析或评论情绪分析

## 常见问题

- **扫码后一直转圈/登录失败**：把 `config/base_config.py` 里 `HEADLESS` 保持 `False`，观察弹出的浏览器窗口是否有滑动验证码，手动过一下再重试
- **想抓更多/更少**：调 `CRAWLER_MAX_NOTES_COUNT`（笔记数量）和 `CRAWLER_MAX_COMMENTS_COUNT_SINGLENOTES`（单篇评论数量）
- **想换关键词重新抓**：直接改 `KEYWORDS` 再跑一次命令即可，不用重新登录

## 安全建议

- 用小号，不用主账号/店铺账号登录
- `CRAWLER_MAX_NOTES_COUNT` 和抓取频率不要设太大，隔几天跑一次比较安全
- 数据仅自己分析用，不对外出售/发布原始数据
