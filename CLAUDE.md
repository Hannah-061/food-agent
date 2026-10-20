# food-agent

一支跑在本機的 Discord agent：指定頻道有人傳食物照片，就用 `claude -p` 判斷食物、估重量與熱量，記進 Excel、存照片，並回覆上傳者與其當日已攝取熱量。

## 開始任何工作前

先讀 `docs/architecture.md`（需求在 `docs/requirements.md`）。架構、欄位、檔名規則、回覆格式都以它為準；要偏離時先跟使用者說明，不要自行改動。

## 四段防護

架構文件第 12 節定義了四段防護，程式裡每一段都要用固定的註解標起來：

```python
# >>> GUARD-3 START：說明這段防護在擋什麼
...
# <<< GUARD-3 END
```

- `GUARD-1`：claude -p 只能讀臨時資料夾，看不到 DATA_DIR
- `GUARD-2`：使用者打的字只當資料，不當指令
- `GUARD-3`：食物名稱當檔名前先清洗
- `GUARD-4`：寫進 Excel 的文字開頭是 `=`、`+`、`-`、`@` 時處理成純文字
- 同一段防護分散在多個檔案時，每一處都要有 START / END 標記。
- 每段旁邊放一行註解起來的「不防護版本」，方便之後教學逐段關掉對照。
- **一開始全部開啟。** 不要自己關掉，也不要做成環境變數開關。

## 不可以 commit 的東西

- 不可以把 `.env` commit 進 git（裡面有 `DISCORD_TOKEN`）。只能 commit `.env.example`。
- `DATA_DIR` 裡的照片和 Excel 也不要 commit（`photos/`、`*.xlsx`、`logs/`）。
- commit 前先看 `git status`，有這些檔案就停下來。
- 日誌、錯誤訊息、測試資料裡都不要印出或寫入 token。
