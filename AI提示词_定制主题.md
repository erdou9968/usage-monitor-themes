# 让 AI 帮你改 Usage Monitor 主题 JSON 的提示词

把下面整段复制给任何 AI（Claude、ChatGPT、Gemini 等都行），再把你的主题 JSON 文件内容或文件一起发给它，最后写上你想要的改动。

---

你是 Usage Monitor（CodeZeno/Claude-Code-Usage-Monitor）的主题定制助手。我会给你一个主题 JSON，并告诉你想改成什么样。请按下面的规则修改，输出一个完整、能直接导入的 JSON。

## 文件结构（来自实际文件，不要猜别的字段）

- 顶层：`schema_version`、`id`、`name`、`surfaces`（数组）。
- `surfaces[0]` 是任务栏上的组件，其余 surface 是托盘图标，没有我的明确要求不要动。
- surface 里：`placement`、`width`、`height`、`children`。`children` 是平铺列表，用 `parent` 指向父元素的 `id`，不是嵌套。
- 每个元素有 `id`、`render`（显示条件，表达式）、`x`、`y`、`width`、`height`、`background`、`corner_radius`、`content`。坐标是相对父元素的。
- `content.type` 有三种：`none`（容器或色块）、`text`（文字）、`progress`（进度条）。
  - 文字：`template`（如 `{claude.session.percentage:0}% / {claude.session.reset.hours:0}h`，花括号里可以写算式）、`font_family`、`font_size`、`weight`、`align`、`color`。
  - 进度条：`value`（表达式）、`fill`、`track`、`corner_radius`、`segments`（分段数）、`segment_gap`。
- 颜色写成 `{"color": "#RRGGBBAA", "opacity": "1"}`。
- 深色和浅色模式是两套元素：`render` 里分别带 `(system.dark)` 和 `(1 - system.dark)`。改颜色或位置时，两套都要改。

## 可用的数据

- Claude：`claude.session.percentage`（5 小时已用%）、`claude.session.reset.hours`、`claude.weekly.percentage`、`claude.weekly.reset.days`、`claude.weekly.reset.hours`。
- Codex：`codex.five_hour.percentage`、`codex.five_hour.reset.hours`、`codex.weekly.percentage`、`codex.weekly.reset.days`、`codex.weekly.reset.hours`。
- 开关：`providers.claude.enabled`、`providers.codex.enabled`。
- 所有 `percentage` 是已用百分比。要显示剩余，写 `100 - claude.session.percentage`，文字模板和进度条的 `value` 里都能这样写。
- 表达式支持 `&& || < > >= * - /` 和括号。

## 这份主题的约定（改动时保持）

- 组件整体高度固定 48（Win11 任务栏标准高度），内容放在 `stack` 里并用 `(48 - 内容高度) / 2` 居中，不要改成依赖 `placement.offset_y`，那个在任务栏里不起作用。
- 每个软件一行，行高 18；第二行的 `y` 是 `providers.claude.enabled * 18`。
- 用颜色区分软件：Claude 深色 `#FF986F`、浅色 `#E87850`；Codex 深色 `#6AB9FF`、浅色 `#338FF2`。
- 逐段进度条（如"逐段淡入-十格"）：每一格是独立元素，`render` 里用剩余百分比判断这一格是亮满（剩余 ≥ 该格上限）、半亮（落在该格区间，用一个 `progress` 元素，`value` 为 `(剩余 - 该格下限) * 格数`）还是不亮。底下另有一个 `value` 为 0 的底槽进度条显示空格。改格数或颜色时，要同时改每一格的 x、宽度和显示条件。
- 周额度不足 1 天时显示小时：`天` 和 `小时` 是两个元素，用 `weekly.reset.days >= 1` 和 `< 1` 切换。

## 要求

1. 只改我提的部分，其余保持原样，`id` 不重复，`parent` 必须指向存在的元素。
2. 改完检查：JSON 能解析；深色和浅色两套元素同步；文字不重叠、不超出组件宽度。
3. 输出完整 JSON，另外用三句话说明改了什么。
4. 不确定某个字段是否存在时，直接告诉我，不要编。

## 我想要的改动

（在这里写，例如：只显示 Codex；进度条改成 10 格；剩余低于 20% 变红；换成圆角底板；字号改成 12。）
