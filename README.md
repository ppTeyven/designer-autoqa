# designer-autoqa · 装修设计器全自动验收工具箱

真机 Chrome 把设计器 HTML 从头到尾实际操作一遍：2D 编辑 / 家具库 / 管线 / 导出 / 3D / 漫游 / 中控 / 多屏幕，
外加切换动效逐帧录像（像人眼一样看 2D↔3D 卡顿和拉伸）。

## 组成

| 文件 | 干什么 |
|---|---|
| `qa.mjs` | 全功能验收主脚本（约 150+ 项，输出 report.md + shots/ + downloads/） |
| `qa_files.py` | 校验导出的 PNG / DXF / PDF / CSV / JSON / 分享 HTML 能不能打开 |
| `qa_furniture.mjs` | 全部家具排成网格分块拍 3D，人工逐件看模型 |
| `qa_styles.mjs` | 10 类家具 × 10 种风格款 + 全部自由模型，分块拍 3D |
| `check.mjs` | 打开 HTML 检查户型：report.json + plan2d.png + overlay.png + view3d.png |
| `walktest.mjs` | 漫游走查：按路线 JSON 真走一遍，验证楼梯 / 扶手 / 门 |
| `record-transition.mjs` | 切换动效逐帧录像：CDP screencast 录 2D→3D→分屏→2D→分屏→3D，输出帧序列 + playback.html 回放页 + metrics.json |
| `serve.py` | 本地静态服务（127.0.0.1:8899，供 record-transition 用） |
| `example-walk-route.json` | walktest 路线示例 |

## 依赖

- Node：本目录 `npm install`（只需 `puppeteer-core`，用本机 Chrome/Edge 真显卡跑，不要用自带 Chromium）
- Python：`pip install pillow ezdxf pypdf`（qa_files.py 用）
- Chrome 或 Edge 已安装；`record-transition.mjs` 需要先起 `python serve.py`（8899）

## 用法

```bash
cd designer-autoqa && npm install

# 1. 模型自查（改完户型/家具先跑这个，秒级）
node check.mjs <设计器.html> <输出目录>

# 2. 全功能验收（约 15~20 分钟；--quick 跳过 PDF 和光线追踪约 8 分钟）
node qa.mjs <设计器.html> <输出目录>
node qa.mjs <设计器.html> <输出目录> --quick
node qa.mjs <设计器.html> <输出目录> --lite --quick   # 电脑性能一般时

# 只跑某一组（2d / 3d / walk / smart / screen，或步骤关键词）
node qa.mjs <设计器.html> <输出目录> --only 2d
node qa.mjs <设计器.html> <输出目录> --only 管线,漫游

# 只重跑上次失败项所在的组
node qa.mjs <设计器.html> <输出目录> --retry <输出目录>/report.json

# 3. 校验导出的文件能不能打开
python qa_files.py <输出目录>/downloads

# 4. 改过家具库 / 3D 建模后
node qa_furniture.mjs <设计器.html> <输出目录>/furn    # 全部家具逐件看
node qa_styles.mjs <设计器.html> <输出目录>/styles     # 风格款 + 自由模型逐件看

# 5. 切换动效逐帧录像（复现"2D 切 3D 那一帧拉伸"）
python serve.py                                        # 另开终端，保持运行
node record-transition.mjs <输出目录>                  # 输出 frames/ + playback.html + metrics.json + summary.json
# 用浏览器打开 <输出目录>/playback.html，选 0.25x 慢放、拖滑杆逐帧看

# 6. 有楼梯必做：漫游走查
node walktest.mjs <设计器.html> example-walk-route.json
```

## 判定规则（qa.mjs）

任何页面报错 = 失败；界面出现 NaN / undefined / Infinity = 失败；
操作后状态没变 = 失败；导出没下载到文件 = 失败；3D 画面空白 = 失败；
夜里中控文字对比度不够 = 失败；1280×720 ~ 1920×1080 下按钮超出 / 重叠 = 失败；
手机尺寸打开平面超出屏幕 = 失败。

`report.md` 每一项 ✅/❌ + 原因 + 截图；有 ❌ 先分清是脚本点错位置还是设计器真问题，
真问题修设计器，**不能带着 ❌ 交付**。自动检查之外一定要人工过一遍 `shots/`。

## 注意事项

- 验收进程默认"低于正常"优先级，电脑照常能用；**不要和视频渲染 / 另一个验收 / 录屏同时跑**，抢显卡会让 3D / 漫游 / 中控几十项一起超时。
- 交付前必须完整跑一遍（可以 `--lite --quick`）。
- record-transition 复现的是"刷新后第一次切换"：脚本会等 3D 就绪 + 预热后才开始点，保证每次条件一致。
