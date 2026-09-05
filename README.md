# context-distill

Claude Code skill：維護每個 feature 的 `TRACKING.md` 工作狀態文件，並在需要時把穩定下來的內容升格成正式文件、把作廢的內容搬去 archive。

這是一份**協定**，不是通用工具。它假設一套固定的 TRACKING.md 骨架（§0–§7、檔頭交接段），只在遵守這套骨架的 repo 裡成立。

## 它管什麼

一條鏈：

```
session ──capture──▶ TRACKING.md（工作狀態）──distill──▶ reference / explanation
                                │
                              compact
                                ▼
                            archive + tombstone
```

TRACKING.md 是給下一個 session 恢復狀態用的，不是給人讀的最終文件。人讀的東西（reference、explanation）由 distill 從它蒸餾出來；它自己則靠 compact 保持可載入的大小。

## 三個模式

| | capture | distill | compact |
|---|---|---|---|
| 觸發 | 手動：session 收尾或壓縮前 | 手動 | 手動 |
| 輸入 | 還在 context 裡的 session | TRACKING 現況 | `superseded`／`closed`／`promoted` 標記 |
| 做什麼 | 新增記錄 | 換讀者 | 搬家 |
| 不變式 | session 裡說過的不丟；既有記錄只 supersede 不刪 | §1／§2／§4／檔頭不變；只寫 §6 與標記行 | 知識總量不變，只換位置 |

三者不合併：每個模式守一條別的模式必然會破壞的承諾。順序上 distill 在前 compact 在後（前者產生 `promoted`，後者消耗它），但 compact 只看標記不做判斷，任何時候跑都安全。

## 記錄類型

| 類型 | 位置 | 寫入政策 |
|---|---|---|
| 交接段 | 檔頭 | 重寫，只寫現在；**只放指標不放內容** |
| 已知事實（含負向事實） | §1 | 追加＋supersede；§1 是暫存區，穩定後升格搬走 |
| ADR | §2 | 追加＋supersede |
| Open Questions | §4 | 狀態機：開 → closed → 索引一行 |
| 方法論 | §6 | 只由 distill 寫入 |

節號是位址不是閱讀順序：§1.n、ADR-n、OQ-n 永不重用、永不重排，跨 feature 引用直接掛在節號上。

## 標記

模式之間只靠三種標記溝通，拼法固定、各佔一行：

```
superseded → §1.14 ｜ 2026-09-10
closed → §1.74 ｜ 2026-08-23
promoted → docs/charge/pricing-reference.md ｜ 2026-09-06
```

`grep -nE '(superseded|closed|promoted) → '` 就是 compact 的候選清單。沒有這三行的記錄對 compact 就是活的。

compact 搬走整個單位後在原位留 tombstone（編號、原因、指向、脈絡、日期），指向它的索引與跨檔引用不需要更新，多一跳即可解析。

## 幾條反直覺的規則

- **原話照留，推理壓縮**：使用者說的、決定的、約束的用引號保留；模型自己的推理只留結論。
- **不補理由**：ADR 沒寫理由就標 `[理由未在紀錄中出現]`，不替使用者推一個。
- **不為困難開區段**：撞過的牆拆成「學到的事實（含已否決的假設）」進 §1，處理過程丟掉。
- **不問**：先假設再行動，假設寫進輸出。
- **不整份載入**：交接段 → §4 → 需要時才讀對應的 §1 小節。

規則的理由都寫在 SKILL.md 裡，因為這些規則跟模型的預設方向相反，沒有理由就不會泛化。

## 安裝

```
cp -r context-distill ~/.claude/skills/
```

在 Claude Code 裡用自然語言觸發：「這個 session 結束了幫我記一下」（capture）、「把穩定的整理成正式文件」（distill）、「TRACKING 太長了瘦一下」（compact）。日常提到 ADR 編號、OQ 結案不會觸發。

## 狀態

試用中。三個模式目前全部手動，刻意不掛 hook——文件格式還在調，自動觸發只會把未定型的行為固定下來。

已知的下一步：試用期觀察標記格式、原話保留、「不補理由」三條的漂移情況；若標記穩定，compact 這段（純機械、且是唯一動 TRACKING 的破壞性操作）會改寫成 script，SKILL.md 只留判斷。

## 設計筆記

- TRACKING.md 在 Diátaxis 的上游：它是工作狀態，不是四種文件類型之一。distill 只產出 reference 和 explanation 兩格；how-to 在這個領域的成熟形態是工具（script、command、skill）而不是文件，tutorial 沒有來源。
- `superseded`／`accepted` 來自 ADR 慣例；`closed` 來自 issue tracker；`tombstone` 借自分散式資料庫的刪除標記；`promoted` 借自部署的 staging → production，因為 §1 被定位成暫存區。
