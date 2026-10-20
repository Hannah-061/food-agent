# Discord Bot 設定步驟（macOS）

這份文件帶你：在 Discord Developer Portal 建立 bot → 打開 Message Content Intent → 拿到 bot token → 把 bot 邀進你自己的 Server。

請先做完 [discord-server-setup.md](discord-server-setup.md)（你要先有 Server 和頻道）。

名詞：

- **Developer Portal（開發者後台）**：Discord 官方給開發者管理程式的網站，網址是 `https://discord.com/developers/applications`。用你平常的 Discord 帳號登入即可。
- **Application（應用程式）**：你在後台建立的「專案」，bot 是裡面的一個部分。
- **bot（機器人帳號）**：Discord 上一個由程式操作的帳號。agent 就是用它收發訊息。
- **bot token**：bot 的**登入密碼**。程式拿著它才能登入成 bot。

---

## ⚠️ 先看這一段：bot token 的規則

**bot token 絕對不能：**

- ❌ 貼給任何人（包含同學、朋友、老師、客服）
- ❌ **貼進這個對話**（包含跟 Claude 的對話）
- ❌ 貼到截圖、螢幕分享、Discord 頻道、社群貼文
- ❌ **commit 進 git**，或傳到 GitHub

**為什麼：** 任何人拿到 token，就能完全冒充你的 bot：讀頻道、發訊息、做任何 bot 有權限做的事。

**token 只能放在：** 專案資料夾裡的 `.env` 檔，`DISCORD_TOKEN=` 後面。`.env` 已經被 `.gitignore` 排除，不會被 commit。

**萬一外洩了：** 馬上回到後台，按 **Reset Token**（步驟見下面第三節），舊的 token 立刻失效；再把新的填進 `.env`。

> 需要 Claude 幫你設定時，請只說「token 已經放進 `.env` 了」，**不要把 token 內容貼出來**。

---

## 一、建立 Application 與 Bot

1. 用 Safari 或 Chrome 打開 `https://discord.com/developers/applications`，用你的 Discord 帳號登入（第一次可能要同意服務條款）。
2. 右上角點 **「New Application」**（藍色按鈕）。
3. 「NAME」輸入名稱，例如 `food-agent`，勾選同意條款，點 **「Create」**。
4. 你會進入這個 Application 的頁面。**左邊選單**有很多項目（General Information、Installation、OAuth2、Bot…）。
5. 點左邊的 **「Bot」**。新的 Application 通常已經自動有一個 bot 了。
6. （建議）在 Bot 頁面中，找到 **「Public Bot」** 開關，把它**關掉**。這樣別人不能自己把你的 bot 邀去別的 Server。
7. 往下找 **「Requires OAuth2 Code Grant」**，保持**關閉**。

---

## 二、打開 Message Content Intent（一定要）

**為什麼一定要：**

- **Intent** 是 Discord 的「資料權限開關」。有些資料 Discord 預設不給 bot。
- 「訊息內容」就是其中之一：**沒打開的話，bot 看得到有人發了訊息，但讀不到訊息裡的文字。**
- 我們的 agent 需要讀文字，因為：
  1. 照片旁邊的補充說明（例如「只吃了一半」）要一起交給 Claude
  2. 使用者回覆來**更正**時，更正的內容就是文字
- 沒打開的話，補充說明和更正功能都不會動，bot 啟動時也可能直接報錯。

**怎麼打開：**

1. 還是在左邊的 **「Bot」** 頁面，往下捲到 **「Privileged Gateway Intents」** 區塊。
2. 找到 **「MESSAGE CONTENT INTENT」**，把右邊的開關**打開**（變成藍色或綠色）。
3. 另外兩個（Presence Intent、Server Members Intent）**保持關閉**，我們用不到。
4. 頁面下方會跳出「有未儲存的變更」的提示，點 **「Save Changes」**。

---

## 三、拿到 bot token

1. 還在 **「Bot」** 頁面，往上找到 **「TOKEN」** 區塊。
2. 點 **「Reset Token」**（第一次可能顯示「Copy」；若是 Reset，會問你確定嗎，按 **「Yes, do it!」**，並可能要求輸入密碼或兩步驟驗證）。
3. 畫面會顯示一串很長的字，**只會顯示這一次**。立刻點 **「Copy」**。
4. **不要**貼到任何對話或文件。直接去做下一步：
   1. 打開專案資料夾裡的 `.env` 檔（如果還沒有，複製 `.env.example` 並改名成 `.env`）
   2. 找到 `DISCORD_TOKEN=` 這一行，把剛剛複製的內容**直接貼在等號後面**，不要加引號、不要有空白
   3. 存檔
5. 貼好之後，**清掉剪貼簿**：隨便複製一個不重要的字（例如在別處選一個字按 ⌘C），避免之後不小心貼到別的地方。

如果你忘了複製、畫面已經關掉：回到 Bot 頁面再按一次 **Reset Token**，會產生新的（舊的作廢）。

---

## 四、決定 bot 的權限

bot 在 Server 裡能做什麼，由「權限」決定。**只給需要的，其他不要給。**

這個 agent 在頻道裡需要的權限只有四個：

| 權限（英文／中文） | 為什麼需要 |
|---|---|
| View Channels／檢視頻道 | 看得到那個頻道 |
| Send Messages／傳送訊息 | 發「處理中」、結果、更正回覆 |
| Read Message History／讀取訊息歷史 | 使用者「回覆」某則訊息來更正時，要能找到被回覆的那則 |
| Attach Files／附加檔案 | 保險用；這一版不會主動傳檔案，但常見設定會一併勾上 |

**不要給：** Administrator（管理員）、Manage Server、Manage Channels、Manage Messages，以及所有「成員管理」的權限。bot 用不到，給了反而是風險。

> 編輯自己發的訊息（把「處理中」改成結果）不需要額外權限。

---

## 五、把 bot 邀進你自己的 Server

1. 在後台左邊點 **「OAuth2」**。
2. 往下捲到 **「OAuth2 URL Generator」** 區塊。
3. 在 **「SCOPES」**（範圍）裡勾選 **`bot`**。**只勾這一個。**
4. 勾選之後，下面會出現 **「BOT PERMISSIONS」** 區塊。勾選這四個：
   - **View Channels**
   - **Send Messages**
   - **Read Message History**
   - **Attach Files**
5. 在頁面最下面有 **「GENERATED URL」**，是一條很長的網址。點右邊的 **「Copy」**。
   - 這條網址**不是**機密，可以安全使用（它只是邀請連結，不含 token）。
6. 把網址貼到瀏覽器的網址列，按 Enter。
7. Discord 會問「要將 `food-agent` 加到哪裡？」，在下拉選單選**你自己的那個 Server**（只會列出你有管理權限的 Server）。
8. 點 **「繼續」**，檢查權限清單是否只有上面四個，沒問題就點 **「授權」**（Authorize）。
9. 可能會有「我是人類」的驗證，照做。
10. 看到「已授權」就完成了。

---

## 六、確認 bot 進來了

1. 回到 Discord，進入你的 Server。
2. 看右邊的**成員清單**，會看到一個有 **「BOT」** 標籤的 `food-agent`，旁邊顯示**離線**（灰色）。
   - 離線是**正常的**：agent 程式還沒啟動，bot 就不會上線。程式跑起來之後才會變成上線（綠色）。
3. 確認 bot 看得到 `# food-photos`：對頻道按右鍵 → **「編輯頻道」** → **「權限」**，若是公開頻道通常預設就看得到，不用改。

---

## 常見問題

| 狀況 | 怎麼辦 |
|---|---|
| 邀請時下拉選單找不到我的 Server | 確認你登入的是建立 Server 的那個 Discord 帳號；Server 要是你建的或你有「管理伺服器」權限 |
| 之後程式啟動報錯 `Privileged intent provided is not enabled` | 第二節沒存好：回到 Bot 頁面，確認 Message Content Intent 開著，並按了 Save Changes |
| 之後程式啟動報錯 `Improper token` / 登入失敗 | `.env` 的 `DISCORD_TOKEN` 貼錯、多了空白或引號；重新 Reset Token 再貼一次 |
| bot 在線，但對照片沒反應 | 檢查 `.env` 的 `CHANNEL_ID` 是否為 `# food-photos` 的編號；bot 是否有該頻道的 View Channels 與 Send Messages |
| 我不小心把 token 貼出去了 | 馬上到 Bot 頁面按 **Reset Token**，再把新的填進 `.env` |

---

## 完成檢查

- [ ] 我在 Developer Portal 建好 Application 和 bot
- [ ] Message Content Intent 已打開並按了 Save Changes
- [ ] Public Bot 已關閉
- [ ] token 已經直接貼進 `.env`，**沒有**貼在別處
- [ ] bot 已用 `bot` scope 與四個權限邀進我的 Server，成員清單看得到它
- [ ] `.env` 有 `DISCORD_TOKEN` 和 `CHANNEL_ID`（`CLAUDE_MODEL`、`DATA_DIR` 之後再填）
- [ ] `git status` 看不到 `.env`
