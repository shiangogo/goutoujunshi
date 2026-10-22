# ChatLab聊天紀錄分析適配

## 定位

ChatLab負責匯入、檢索、統計和返回帶編號的聊天證據；狗頭軍師負責情緒承接、事實／推測拆分、關係判斷與行動建議。適配不賦予狗頭軍師直接讀取或匯出LINE、微信、QQ 等應用程式資料庫的能力。

上游契約可能變化，運行前以 `chatlab manifest` 為準：

- [ChatLab外部Agent說明](https://github.com/ChatLab/ChatLab/blob/main/docs/cn/ai/external-agent.md)
- [chatlab-analyze-cn](https://github.com/ChatLab/ChatLab/blob/main/skills/chatlab-analyze-cn/SKILL.md)
- [chatlab-import-cn](https://github.com/ChatLab/ChatLab/blob/main/skills/chatlab-import-cn/SKILL.md)

## 先鎖定說話人

分析前建立並顯示固定映射，例如“使用者＝member 17／右側藍色，對象A＝member 42／左側白色”。沒有確認前不解釋關係：

- ChatLab或匯出文件優先使用 sender／member ID、帳號和會話元數據；絕不根據左右位置、語氣、性別或“誰更像使用者”判斷。左右和顏色會隨應用、匯出方式與截圖者變化。
- 截圖或元數據缺失時，只問一個最小問題，請使用者確認哪一側或哪個暱稱是自己；一年記錄、跨平台記錄再抽 2–3 條帶時間與短摘錄的錨點核對，衝突時暫停分析。
- 跨文件、跨年份和分塊查詢沿用同一映射；只有格式或帳號改變時才重新確認，不能因內容、稱謂或代詞靜默翻轉。

## 已經匯入ChatLab

先檢查當前命令契約和候選會話：

```bash
chatlab --help
chatlab manifest
chatlab sessions list --format json
```

存在多個會話、同名成員或“me”身份不明時讓使用者消歧。先使用最窄的專用查詢，並限定對象和時間：

```bash
chatlab messages between --member me --member <member> --session <session-id> --last 90d --format agent
chatlab messages search "<keyword>" --session <session-id> --format agent
chatlab messages context --id <message-id> --session <session-id> --window 10 --format agent
```

結果不足時再查統計或上下文，最後才使用只讀SQL。不得使用 `--raw`、修改資料庫、洩露完整聊天或為了“更全面”無邊界翻頁。

## 使用者提供已匯出文件

先確認一個準確的絕對路徑；存在多個候選文件時不猜。ChatLab CLI未安裝時只給官方安裝方式，不擅自安裝。

先只讀預覽：

```bash
chatlab import "/absolute/path/to/chat-export.json" --dry-run --json
```

向使用者報告識別格式、目標會話、新增與重復訊息數，不引用訊息正文。使用者已經明確要求匯入且預覽結果安全時，可使用相同路徑正式匯入；若使用者只要求預覽則停止。格式無法識別時查看 `chatlab formats`，不要假裝支持。

ChatLab只能處理使用者已經取得的匯出文件。使用者說“幫我匯出 LINE 或微信聊天”時，明確說明當前適配只能分析已匯出的文件，不調用第三方解密工具、不繞過應用權限。

## 關係趨勢與K線

[Relationship Candlestick Lab](https://github.com/ZhenyuanPAN822/relationship-candlestick-lab) 可作為可選探索層。只吸收按時間聚合、關鍵轉折、事件歸因和代表性訊息編號；K線價格、MACD、RSI、KDJ、均值回歸和固定基線不是經過驗證的關係測量，不能據此判斷愛意、忠誠或分手概率，也不能寫入穩定檔案。使用者想看趨勢時優先展示可解釋的主動頻率、回覆間隔、邀約兌現、衝突與修復事件，並顯式標註模型評分的主觀性。

## 回答與記憶邊界

回答開頭說明實際查詢的會話和時間範圍。引用 `[#1021]` 等證據編號，分開寫：

1. 可觀察事實：主動次數、兌現、時間間隔、明確原話。
2. 合理解釋：可能的關係含義和其他解釋。
3. 關鍵未知：線下互動、沒有說出口的意圖、應用外事件。
4. 建議：低成本驗證動作、觀察窗口和停止條件。

原始聊天、長摘要和統計表不進入長期記憶。首次同意後，也只允許把關鍵事件的簡短摘要、日期和證據編號寫入 `event`；由ChatLab或模型得出的性格與意圖只能寫入 `hypothesis`，不能升級為對象事實。

