# AI 電話助理研究：用一般門號做出能跟 Whoscall 競爭的「個人 AI 助理」（2026-10-23）

> 前一份：`whoscall-research.md`（Whoscall 的公司、營收、資料庫與弱點）
> 這份回答三件事：市面上別人怎麼做、技術上一般門號怎麼接上 AI、我們該做哪些功能。
> 標示規則：**【已查證】** 有官方或可信來源；**【二手】** 只有媒體／部落格；**【未查到】** 找不到可靠資料，要再打客服或實測確認。

---

## 0. 先講結論（給沒時間看完的人）

1. **最大的威脅不是 Whoscall，是 iPhone 自己。** Apple iOS 26 的「篩選來電」（Call Screening）和 Hold Assist，官方語言清單已包含「Mandarin Chinese (Taiwan)」，也就是台灣 iPhone 使用者免費就有「AI 先問你是誰、找我什麼事」。所以我們不能只做「代接問來意」，一定要做到 iPhone 內建做不到的部分：**你不在手機前面也能處理完**（例如開車時）、**把內容整理成訂單或待辦**、**推播到 LINE**、**台語／英文客人**。
2. **一般門號接 AI 的唯一實際做法是「電信端條件轉接」**：使用者在自己手機設定「無人接聽／忙線／不能接通時轉接」到我們的台灣號碼，電話就到我們的雲端 AI。iOS 和 Android 都不讓一般 App 直接聽或錄通話聲音，所以「純 App」做不到 AI 代接。Truecaller、Hiya 也是這樣做。
3. **最大的未知數是轉接費用和台灣號碼取得**：轉接那一段通常算使用者「自己打出去」的電話費（依方案費率計算），三家電信的官方費率頁面我都查不到明確說明，要打客服確認。轉接目的地必須是**台灣號碼**（轉到國外號碼會變國際電話），Twilio 有台灣號碼但要附身分證件和地址，實際有沒有庫存要開帳號查。
4. **建議 MVP 前三名**：①AI 代接問清楚來意 ②陌生來電摘要推播（含一鍵回覆）③客人訂單／行程自動記錄。「隱私優先（不上傳通訊錄）」直接當成產品的基本原則，從第一版就這樣設計，不算在功能名額裡。

---

## 1. 市面上的 AI 電話助理怎麼做

### 1.1 總覽表

| 產品 | 怎麼接上電話 | 主要功能 | 價格 | 台灣能不能用 |
|---|---|---|---|---|
| **Truecaller AI Assistant** | 電信端條件轉接到 Truecaller 的號碼（業界通用做法，但 Truecaller 官方設定說明頁我抓不到，**【二手】**） | AI 代接、問來意、即時逐字稿、可以按按鈕叫對方多講一點或標記詐騙；2024 年起可以用「自己聲音的 AI 複製版」接聽（跟 Microsoft Azure AI Speech 合作） | App Store 列出 Assistant 月費 US$4.99～9.99（不同時期和地區不一樣）；印度 2023 年推出時 INR149/月 | **不行。** App Store 寫明只有美、加、澳、智利、南非、瑞典、以色列，後來加上印度 Android **【二手】** |
| **Google Pixel Call Screen** | 系統內建電話 App（Google 自己的 App，才有權限碰到通話聲音） | 自動或手動篩選，AI 問來意並顯示逐字稿 | 免費（買 Pixel 就有） | **不行。** Google 官方說明頁：自動篩選只有澳、加、愛爾蘭、英、美；手動篩選多了法、德、印度、義、日、西，**沒有台灣** **【已查證】** |
| **Google Hold for Me** | 系統內建 | 打客服被要求「請稍候」時，AI 替你等，真人接起來才叫你 | 免費 | 早期只支援英文、免付費號碼；2026 年最新國家清單**【未查到】**，台灣大概率沒有 |
| **Apple iOS 26 篩選來電（Call Screening）** | 系統內建電話 App | 沒存過的號碼打來時，iPhone 先問對方姓名和來意，轉成文字顯示，你再決定要不要接；選項有「永不／詢問來電原因／靜音」 | 免費 | **可以。** Apple 官方功能表的語言清單有「Mandarin Chinese (Taiwan)」**【已查證】** |
| **Apple Hold Assist** | 系統內建 | 跟 Hold for Me 一樣，幫你等客服 | 免費 | 語言清單有「Mandarin Chinese (Taiwan)」**【已查證】** |
| Apple 其他相關 | 系統內建 | 「即時語音信箱」（Live Voicemail）、通話錄音、通話逐字稿，也都支援台灣國語；Apple Intelligence 的通話摘要支援繁體中文 | 免費（摘要需要支援 Apple Intelligence 的機型） | **可以** **【已查證】** |
| **Samsung Galaxy AI 通話助理（Call Assist）** | 系統內建 | 通話即時翻譯（Live Translate，支援繁中—台灣／香港）、通話逐字稿（One UI 7 支援台灣中文）；「文字通話」（Text Call／Bixby Text Call）讓你打字、AI 唸給對方聽 | 免費（買 Galaxy 高階機） | 翻譯和逐字稿**可以**；「AI 自己問來意、自己處理」的代接功能，中文支援度**【未查到】** |
| **Hiya AI Phone** | 電信端轉接（App Store 說「透過你的電信商設定轉接，App 會帶你操作」；Android 版會自動判斷電信商並設定） | AI 篩選陌生來電並告訴你對方姓名和來意、通話摘要和逐字稿、即時詐騙偵測、**合成語音（深偽）偵測**、可以替你打短電話確認資訊和排行程 | 免費試用 14 天，之後約 US$9.99/月 **【二手】** | **不行。** 官網寫「Currently only available in the US」（Android 說明寫美國或加拿大）**【已查證】** |
| **中國：中國移動「AI 秘書」**（廣東移動，2025-05） | 電信網路端（電信商自己做，不用轉接設定） | 你拒接、關機、沒訊號時由 AI 接聽，錄音並轉成文字，用簡訊把錄音和文字傳給你 | **【未查到】** | 不行（只限中國移動用戶） |
| **中國：中國電信「AI 代接」** | 電信網路端 | 官方 App 描述「AI 代接：智慧通訊助理，重要來電不漏接」 | **【未查到】** | 不行 |
| **中國：騰訊／小米／360** | — | 騰訊雲有給企業用的「AI 客服語音代理」，不是個人代接；小米 2019 年 MIUI 11 有「AI 通話助理」（把對方的話轉成文字、讓你點選回覆，不是 AI 自己跟對方對話），目前澎湃 OS 有沒有新版代接**【未查到】**；360 只查到 2018 年一般語音助理新聞，跟代接無關 | — | 不行 |
| **韓國 SK Telecom A.（에이닷）電話／原 T Phone** | 電信商自家電話 App（T Phone 改名 A. Phone），SKT 用戶可以用網路端功能 | 通話錄音（iPhone 也可以）、AI 通話摘要、即時口譯、**AI 詐騙指示燈**（陌生來電時顯示「釣魚警告／垃圾來電警告／疑似垃圾」，由 SKT 的 AI 即時分析號碼異常模式） | 基本免費，但摘要有次數上限：2024 年報導每月 30 通；2025 年 iPhone 17 促銷寫「基本 350 次」再送 1,000 次（限期）。目前正式方案**【未查到】** | 不行（韓國門號） |
| **台灣：中華電信** | — | 123 客服用語音 AI；有說過要讓 AI 對話系統幫企業接單、訂位；**沒有給一般用戶的 AI 代接** | — | **沒有類似的個人服務**（截至搜尋結果）**【二手】** |
| **台灣：台灣大哥大** | — | 發表了能辨識台語、英語、客語混說的語音辨識模型，示範用在銀行客服；**沒有個人 AI 代接** | — | 同上 |
| **台灣：遠傳** | — | 主打「AI 防詐平台」，宣稱擋詐騙電話、攔截 9 成以上詐騙簡訊；有「遠傳智靈」企業 AI；**沒有個人 AI 代接** | — | 同上 |
| **Whoscall（參考）** | App（iOS 用 CallKit 和 Live Caller ID Lookup） | 2025-09 改版主打 AI 防詐；「詐騙訊息檢查」可以丟截圖給 AI 判斷；**「語音示警」先在泰國上線**（來電時用語音唸出號碼和可疑用途，主打給視障者和 Grab 司機、外送員開車時用），說之後會來台灣，**時程未公布** | 見前一份報告 | 是我們的直接對手 |

### 1.2 從這些產品學到的事

- **兩條路線**：(A) 手機廠商／電信商自己做（Apple、Google、Samsung、SKT、中國移動），因為他們能碰到通話聲音或電信網路，**我們做不到這條**；(B) 第三方 App + 電信條件轉接 + 雲端 AI（Truecaller、Hiya），**我們只能走這條**。
- **第三方路線在台灣目前是空白**：Truecaller Assistant、Hiya AI Phone 都沒有開放台灣，三家電信也沒有個人 AI 代接。
- **Whoscall 已經在注意「司機」這個族群**（泰國的語音示警），但他們做的是「唸出來電是誰」，不是「AI 幫你接、幫你記」。
- **iOS 26 的篩選來電是免費的基本款**，台灣用戶很快會習慣「陌生來電先被問來意」。我們要賣的是它做不到的事（見第 3 節）。

---

## 2. 技術：一般門號怎麼接上 AI

### 2.1 架構圖（文字版）

```
來電者 ──打給──> 使用者自己的門號（中華/台哥大/遠傳）
                    │
                    │ 使用者沒接 / 忙線 / 沒訊號
                    ▼  （電信端條件轉接，使用者自己設定一次）
              我們的台灣號碼（Twilio 或台灣 VoIP 業者）
                    │
                    ▼
              雲端：語音 AI（聽 → 理解 → 回話），同時做逐字稿、摘要、詐騙判斷
                    │
                    ▼
        推播到使用者的 App / LINE：「09xx 是客人王先生，11/2 早上 6 點桃機送機 4 人」
```

重點：**使用者完全不用換號碼、不用申請新門號。** 客人照樣打原本的號碼；使用者有空就自己接，沒空才換 AI 接。

### 2.2 台灣電信的條件轉接

**設定代碼**（這是 GSM 國際標準代碼，台灣三家都是 GSM 系統，大致通用；**各家實際支援程度要實測**）：

| 情況 | 啟用 | 取消 | 適合我們嗎 |
|---|---|---|---|
| 無人接聽轉接 | `**61*轉接號碼#`（標準格式也可加響鈴秒數：`**61*號碼**秒數#`，常見 5～30 秒，**台灣三家是否支援自訂秒數【未查到】**） | `##61#` | **主力**：響幾聲沒接才給 AI |
| 忙線轉接 | `**67*轉接號碼#` | `##67#` | 要：正在講電話時，第二通給 AI |
| 不能接通（關機、沒訊號、飛航） | `**62*轉接號碼#` | `##62#` | 要：司機在地下停車場、睡覺關機時 |
| 無條件轉接 | `**21*轉接號碼#` | `##21#` | **不要用**，而且要教育使用者避開（見下方風險） |
| 一次設好「沒接＋忙線＋不能接通」 | `**004*轉接號碼#` | `##004#` | 最方便，但**三家支不支援【未查到】** |
| 查詢狀態 | `*#61#`、`*#67#`、`*#62#` | — | App 可以引導使用者查 |

iPhone 也可以在「設定 → 電話 → 來電轉接」設定，但那個選單只有「無條件轉接」，**不適合**；條件轉接要用撥號代碼。

**使用者可以自己設定嗎？** 可以，撥代碼就好，不用到門市。但要注意：手機必須在有訊號時設定（飛航模式下設定不會送到電信端）**【二手】**。

**要付多少錢？**（這是目前最大的未知數）
- 一般說法：**轉接功能本身免月租，被轉出去的那段通話算使用者自己撥出的電話費**，依使用者方案的語音費率計算 **【二手】**。也就是說，來電者付「打到使用者」的錢，使用者付「轉到 AI 號碼」那段的錢。
- 中華電信官網的「指定轉接」頁面是**市話**的服務：月租 30 元，並寫「客戶須負擔轉接至另一通話號碼之通信費」**【已查證，但是市話不是手機】**。
- 一篇比較文章寫台灣大哥大「功能月租費免費，通信費比照主要資費之語音費率計收」**【二手，文章註明以業者公告為準】**。
- 遠傳：**【未查到】**官方行動轉接費率。
- **實務影響**：如果我們的 AI 號碼是「網外手機號碼」或「070 網路電話」，使用者就付網外或撥 070 的費率；吃到飽方案通常包含網外分鐘數，**但轉接通話算不算在吃到飽內，要逐家打客服確認**。這直接影響產品能不能成立，**建議第一步就打 123（中華）、188（台哥大）、888（遠傳）問清楚**。

**風險：`**21` 是詐騙常用手法。** 刑事局案例：詐騙集團誘導民眾輸入 `**21*` 加指定門號，把所有來電（包括銀行驗證電話）轉走，盜刷 260 萬 **【二手，有新聞來源】**。所以：
- 我們的 App **只用 `**61`/`**67`/`**62`**，絕對不用 `**21`；
- 設定時要清楚顯示「轉接到 0X-XXXX（XX 公司的 AI 號碼）」，並提供一鍵取消；
- 這也是行銷時要面對的信任問題：「叫我輸入轉接代碼」聽起來很像詐騙，App 內引導和官方說明要做得非常清楚。

### 2.3 轉接目的地：去哪裡拿台灣號碼

轉接一定要轉到**台灣號碼**，轉到美國號碼會變國際電話，使用者帳單會爆掉。

| 來源 | 台灣號碼 | 條件 | 費用 | 判斷 |
|---|---|---|---|---|
| **Twilio** | 官方列出台灣有市話（local）、手機（+8869）、免付費（+88680）三種 | 個人：身分證或居留證（護照當第二證件）＋號碼所在地區的地址證明（不收郵政信箱）；公司：公司登記＋負責人證件。免付費號碼**不開放個人** | 號碼月租：官網只寫國際號碼「Starting at US$1.15/月」，台灣各類號碼的實際月租要下載價目 CSV **【未查到確切數字】**；接聽：SIP／App 端 US$0.004/分鐘，**用台灣市話或手機號碼接聽的每分鐘費用官網沒列出**；撥出到台灣市話 US$0.1196/分、手機 US$0.1985/分 | **最快可以做出原型**。但每個使用者要一個號碼的話，證件、庫存和成本要先確認。台灣不支援雙向簡訊 |
| **Vonage / Telnyx** | **【未查到】**台灣號碼供應資訊 | — | — | 要問業務 |
| **台灣 070 網路電話業者**（例如是方電訊 Chief） | 070 號碼可以跟一般市話、手機互通，用 SIP 協定 | 2005 年制度：資本額 5 億以上向主管機關申請，以下向第一類電信申請；個人可以透過部分業者的 App 申請 070 號碼 | **【未查到】**現在的批發／SIP trunk 價格 | **長期最划算的方向**：本地業者、本地號碼、在地客服。缺點是要談合作，非工程師出身比較難自己弄 |

**設計建議：一個 AI 號碼可以服務很多使用者嗎？** 理論上可以：電信轉接時，我們的系統通常收得到「原本被撥打的號碼」（技術上叫 Diversion header），就能知道這通是哪個使用者的客人。**但台灣電信轉接時會不會帶這個資訊【未查到】，一定要實測**。如果不會帶，就得每個使用者配一個號碼，成本較高。

### 2.4 AI 即時語音對話的技術組合

**兩種做法：**

| 做法 | 說明 | 優點 | 缺點 |
|---|---|---|---|
| **語音直接對語音**（OpenAI Realtime API、Google Gemini Live API） | 一個模型直接聽聲音、直接講話 | 延遲低、語氣自然、開發簡單 | 比較貴；台語能力未知；比較難逐字檢查它講了什麼 |
| **三段串接**（語音轉文字 STT → 大型語言模型 LLM → 文字轉語音 TTS） | 分三個零件 | 每段可以換最適合台灣的（例如台語模型）；文字留存方便做摘要和稽核 | 延遲要調校；開發比較複雜 |

**延遲目標**：業界標準是從對方講完話到 AI 開口 **800 毫秒以內**才像真人；Salesforce 研究團隊的三段串接最佳成績約 729～755 毫秒 **【已查證：論文／技術文章】**。最大的延遲來源通常是「判斷對方講完了沒」，調太快 AI 會插話。

**中文和台語的表現：**
- **台灣國語**：Apple 自己的篩選來電已經支援台灣國語，代表技術上成熟。各家雲端 STT 都支援 zh-TW，但**我沒查到公開的台灣國語辨識準確率比較數據**，要用真實客人的錄音自己測（尤其是國台語混說、地名、航班號碼）。
- **台語**：聯發創新基地（MediaTek Research）2026-03 發表 **Breeze ASR 26**（以 Whisper 為基礎、約 1 萬小時台語語音訓練、包含國台語混用，宣布開源）和 **BreezyVoice 26**（台語文字轉語音）**【二手：新聞報導】**；前一代 Breeze ASR 25 是 Apache 2.0 開源。台灣大哥大也發表了能辨識台語、英語、客語混說的模型（是否對外提供 API **【未查到】**）。**台語是 Whoscall、Apple、Google 都還沒做的差異點**，但成熟度要實測。
- **英文**：給外國旅客用，所有方案都很成熟。

**每分鐘成本估算**（美元，**都是第三方整理的 2026 年數字，彼此不一致，上線前要對官方價目**）：

| 項目 | 估計 |
|---|---|
| OpenAI gpt-realtime 系列（語音對語音，大模型） | 約 US$0.18～0.30/分鐘 |
| OpenAI realtime mini 版 | 約 US$0.06～0.10/分鐘 |
| Google Gemini Live（Flash 原生語音） | 約 US$0.012～0.02/分鐘（多數比較中最便宜） |
| Deepgram Voice Agent（三段打包） | 約 US$0.05～0.08/分鐘（**以連線時間計費，沉默也算錢**） |
| ElevenLabs Agents | 約 US$0.08～0.10/分鐘 |
| 電信端（Twilio 接聽） | SIP 端 US$0.004/分鐘起；台灣號碼接聽費率**【未查到】** |
| 使用者自付的轉接段電話費 | 依使用者方案，**【未查到】** |

**用司機情境試算**（假設：每天 15 通陌生或沒接到的來電、每通 AI 講 45 秒 → 每月約 340 分鐘）：
- 用 Gemini Live 等級：AI 約 US$4～7 ＋ 電信 US$1.5～3 ＝ **約 US$6～10／月（約 NT$190～320）**
- 用 OpenAI 大模型：AI 約 US$60～100 ＝ **約 NT$2,000～3,200／月**，**一般個人方案撐不住**
- 一般非司機使用者（每天 2～3 通）約只有 1/5～1/6 的用量。
- **結論**：要用便宜的語音模型、控制每通長度（AI 問完重點就收線，45～60 秒上限），才能把月費訂在 Whoscall 附近（NT$95/月）到「專業版」（例如 NT$299～399/月）的區間。這是估算，不是報價。

### 2.5 iOS 和 Android 的限制

| | iOS | Android |
|---|---|---|
| App 能不能聽／錄電話聲音 | **不能。** Apple 開發者技術支援在官方論壇直接回答「No」，沒有任何 API 能讓 App 拿到通話錄音 **【已查證】**。iOS 18 起內建電話 App 自己有錄音（會告知對方），但錄音檔不開放給第三方 App | **不能。** Android 10 起禁止 App 用麥克風錄通話；2022-05-11 起 Google Play 禁止用「無障礙 API」錄通話。只有「預載在手機裡的系統預設撥號 App」才拿得到通話聲音 **【二手：多家科技媒體報導 Google 政策】** |
| App 能不能自己接起電話交給 AI | 不能 | 一般 App 不能；就算把我們的 App 設成預設撥號 App，也拿不到通話聲音 |
| 能做的事 | CallKit 號碼辨識（離線號碼包）、Live Caller ID Lookup（雲端即時查詢，Whoscall 已在用）、來電封鎖 | CallScreeningService（判斷要不要擋、顯示來電標籤，但拿不到聲音）、更彈性的來電畫面 |
| 對我們的意義 | **AI 代接一定要走電信條件轉接 → 雲端**，手機端 App 負責：設定引導、推播、摘要、訂單管理、號碼辨識 | 同左。Android 的好處是可以做更即時的來電畫面提示 |

**法律提醒**：台灣《通訊保障及監察法》第 29 條，通話的一方錄音、目的不違法，原則上不違法 **【二手：法律科普文章】**。但 AI 代接是「我們的公司系統」在錄音和處理資料，還會碰到**個人資料保護法**的告知義務。保守做法：AI 開場就說「您好，這是 XXX 的 AI 助理，通話會被記錄並整理給他」。**正式上線前要問律師。** 另外也要避免 AI 假裝是本人（Truecaller 的「自己聲音的 AI」也特別限制開場白自訂，就是為了透明）。

---

## 3. 建議的差異化功能清單（11 項）

> 「Whoscall 為什麼沒做」的判斷依據：Whoscall 的核心是號碼資料庫＋App 內廣告＋訂閱（見前一份報告），iOS 版用 CallKit，沒有電信轉接和語音 AI 的基礎設施，每分鐘的語音 AI 成本也不符合他們 NT$95 月費＋廣告的商業模式。這是「目前沒做」，不代表他們永遠做不到，他們資金和 165 的關係都比我們強。

| # | 功能 | 在做什麼 | 為什麼 Whoscall 做不到或沒做 | 難度 | MVP？ |
|---|---|---|---|---|---|
| 1 | **AI 代接並問清楚來意** | 沒接、忙線、沒訊號時轉給 AI，AI 問「請問怎麼稱呼、有什麼事、方便什麼時候回電」，必要時追問（例如「是送機還是接機？幾點的航班？」） | Whoscall 只「辨識號碼是誰」，不接電話；iPhone 篩選只問一句、你人不在手機前就沒用 | 高（電信轉接＋即時語音 AI＋延遲調校） | **是（核心）** |
| 2 | **陌生來電摘要推播**（含一鍵回覆） | 通話結束 5 秒內推播一段摘要：「王先生，11/2 06:00 桃機送機，4 人 3 箱，請回電」；一鍵回簡訊或一鍵回撥 | Whoscall 沒有通話內容，只能推「這是 XX 公司」；Apple 的摘要只在 iPhone 裡，不會進你的工作流程 | 中 | **是** |
| 3 | **客人訂單／行程自動記錄** | 從對話抽出日期、時間、航班、上下車地點、人數、行李、車型、報價，變成一筆待確認訂單，可以匯出到 Google 日曆、試算表或 LINE | 這是垂直行業需求，Whoscall 是通用防詐 App；Apple、Google 也不會為「接送業」做欄位 | 中（大型語言模型擅長抽資料，難的是地名、航班號碼聽錯） | **是（司機情境的殺手功能）** |
| 4 | **詐騙即時語音警示** | AI 代接時，即時判斷對方話術（假檢警、解除分期、猜猜我是誰、要你按鍵或轉帳）＋偵測合成語音，判斷是詐騙就直接結束通話並推「已擋下疑似詐騙：自稱 XX 銀行」 | Whoscall 靠號碼回報，新號碼第一次打來就抓不到（前一份報告提到誤判問題）；我們看的是「講了什麼」 | 高（要即時判斷、誤判成本高）；**限制：只能分析 AI 代接的那段，使用者自己接的電話 iOS／Android 都聽不到** | 第一版做「AI 代接段的話術判斷」；合成語音偵測放第二階段 |
| 5 | **隱私優先：不上傳通訊錄** | 通訊錄只在手機本機比對；雲端只留最少資料，錄音可以設定「摘要完成就刪除」；不賣資料、不放廣告 | Whoscall 2015 年被抓到上傳通話紀錄，同意條款要求分享聯絡人，免費版廣告多（前一份報告）——這是他們的品牌包袱 | 低～中 | **是（當作產品原則，不算功能名額）** |
| 6 | **情境模式**（開車／睡覺／忙碌） | 一鍵或自動（連上車用藍牙、晚上 3 點後）切換：開車時熟客直接接、陌生人全給 AI；睡覺時只有家人和熟客能響 | Whoscall 只有封鎖或不封鎖；iOS 的專注模式不會幫你接電話 | 低～中（電信轉接設定不能頻繁切換，要用 App 端邏輯配合） | 簡易版可以放 MVP |
| 7 | **AI 知道你的價目表和常見問題** | 使用者上傳價格（例如龜山到桃機多少、深夜加價）、服務範圍，AI 能直接回答「大概多少錢」，但標註「最終由司機確認」 | Whoscall 沒有「代表你說話」的角色 | 中（要防止 AI 亂報價） | 第二階段 |
| 8 | **國語／台語／英文接聽** | 自動判斷對方語言；長輩講台語、外國旅客講英文都能接 | Apple 篩選目前沒有台語；Whoscall 不接電話 | 高（台語靠 Breeze ASR 26 等新模型，要實測） | 英文可以先上（技術成熟），台語第二階段 |
| 9 | **熟客記憶** | 同一個號碼第二次打來，AI 認得：「陳小姐您好，又要從板橋到機場嗎？」並把上次訂單帶出來 | Whoscall 的資料庫是「全世界對這個號碼的評價」，不是「你跟這個人的關係」 | 中 | 第二階段 |
| 10 | **家人防詐守護** | 子女幫長輩開啟：長輩的陌生來電先給 AI，判斷是詐騙就擋下並通知子女 | Whoscall 有家庭方案但只是共享訂閱；沒有「AI 先幫長輩過濾」 | 中（長輩的轉接設定要子女幫忙） | 第二階段（很好的行銷故事，也可能是比較大的市場） |
| 11 | **AI 驗證過的號碼標籤**（減少誤判） | 用 AI 實際對話的結果（「自稱 XX 物流、內容是送貨通知」）去補強號碼標籤，而不是只靠使用者回報 | Whoscall 靠群眾回報、沒有驗證機制，警方號碼曾被標成詐騙（前一份報告） | 中（需要一定的使用者數才有效果，還有隱私設計問題） | 第三階段 |

### 3.1 使用情境舉例（司機與非司機）

- **機場接送司機（使用者本人）**：開車載客中有陌生號碼打來 → 響 3 聲沒接 → AI 接起：「您好，這是 X 司機的 AI 助理，他正在開車。請問是要預約接送嗎？」→ 問完日期、航班、人數 → 下車後手機出現一筆待確認訂單，一鍵回簡訊確認。**安全賣點：開車不用碰手機。**
- **水電師傅、美髮師、小吃店老闆、家教**：工作中雙手沒空，AI 幫你收預約。
- **上班族、業務**：開會中，AI 把重要來電整理好；推銷電話直接擋掉。
- **長輩**：子女幫忙開啟防詐守護。

---

## 4. 建議的 MVP 前 3 名

| 名次 | 功能 | 為什麼 |
|---|---|---|
| **1** | **AI 代接並問清楚來意** | 整個產品的入口和技術核心；台灣第三方市場目前沒人做（Truecaller、Hiya 都沒開台灣，電信沒做）。先做國語，問來意＋必要追問，每通 60 秒內收線控制成本 |
| **2** | **陌生來電摘要推播（含一鍵回覆）** | 這是跟 iPhone 內建篩選拉開差距的關鍵：iPhone 要你當下看螢幕，我們是「你忙完再看就好」。技術難度中等，建在第 1 名之上 |
| **3** | **客人訂單／行程自動記錄** | 使用者自己就是第一個客戶，可以自己測、自己用；先打穩「接送司機」這個小族群，比正面跟 Whoscall 搶通用市場務實。之後再複製到其他「靠電話接單」的行業 |

**從第一天就要做好的基本原則（不算名額）**：隱私優先（通訊錄不上傳、錄音可自動刪除、無廣告）、AI 開場明講自己是 AI、只用 `**61`/`**67`/`**62`、絕不用 `**21`。

**第一版先不要做**：台語、合成語音偵測、熟客記憶、AI 報價。

---

## 5. 下一步要先確認的事（依優先順序）

1. **打三家電信客服**：行動門號 `**61`/`**62`/`**67` 能不能用、能不能設響鈴秒數、轉接到「網外手機號碼」或「070」怎麼計費、吃到飽方案包不包含轉接段。**這題答案決定產品能不能成立。**
2. **開 Twilio 帳號實測**：能不能買到台灣號碼（手機或市話）、要附哪些文件、月租多少；轉接進來時收不收得到「原本被撥的號碼」。
3. **用自己的門號跑一個最小原型**：條件轉接 → Twilio → 語音 AI → LINE 推播摘要。測延遲、國語辨識準確度（地名、航班號碼）、每通成本。
4. **問律師**：AI 代接錄音的告知方式、個資法義務。
5. **問 070 業者**（例如是方電訊）批發或 SIP trunk 價格，當成 Twilio 之外的長期方案。

---

## 來源

**AI 電話助理產品**
- Truecaller Assistant 國家與價格（App Store 定價追蹤）：https://apppricinglab.com/app/apple/448142450
- Truecaller Assistant 印度上線（GSMArena）：https://m.gsmarena.com/truecaller_assistant_india_launch_price_details-amp-59245.php
- Truecaller 2022 推出 AI Assistant（Business Wire）：https://www.businesswire.com/news/home/20220928005210/en/Truecaller-Launches-AI-powered-Assistant-to-Filter-Out-Fraud-and-Scam-Calls
- Truecaller 用自己聲音的 AI 接電話（Android Authority）：https://androidauthority.com/truecaller-ai-assistant-voice-clone-3445591
- Google Call Screen 官方國家清單：https://support.google.com/phoneapp/answer/9118387?hl=en
- Pixel Call Screen 擴大國家（Android Authority）：https://www.androidauthority.com/google-pixel-call-screening-more-countries-india-3400823/
- Hold for Me（Android Authority）：https://androidauthority.com/hold-for-me-google-assistant-pixel-3249821
- Apple iOS 功能可用地區與語言（官方）：https://www.apple.com/ios/feature-availability/
- iOS 26 Hold Assist 實測（Tom's Guide）：https://www.tomsguide.com/phones/iphones/i-tried-hold-assist-in-ios-26-and-it-solves-one-of-the-most-annoying-things-about-phone-calls-heres-how-it-works
- iOS 26 電話 App 新功能（Neowin）：https://www.neowin.net/news/exploring-ios-26-10-new-features-in-the-phone-app/
- iOS 篩選來電繁中介紹（瘋先生）：https://mrmad.com.tw/iphone-call-screening
- Samsung Live Translate 支援繁中（Samsung 官方支援頁）：https://www.samsung.com/sg/support/mobile-devices/how-to-use-live-translate-for-phone-calls-on-the-galaxy-s24/
- Samsung One UI 7 通話逐字稿語言（Neowin）：https://www.neowin.net/news/samsungs-one-ui-7-call-transcript-feature-has-an-edge-over-apples-and-pixels-versions/
- Bixby Text Call（9to5Google）：https://9to5google.com/2022/11/28/samsung-bixby-text-call/
- Hiya AI Phone 官網：https://www.hiya.com/products/apps/hiya-ai-phone
- Hiya AI Phone App Store：https://apps.apple.com/us/app/-/id6474703665
- Hiya Android 篩選設定說明：https://apphelp.hiya.com/en/articles/32-how-to-use-call-screener-on-android
- Hiya AI Phone 發表（Help Net Security）：https://www.helpnetsecurity.com/2025/01/28/hiya-ai-phone/
- 中國移動 AI 秘書（IT 之家）：https://www.ithome.com/0/854/053.htm
- 中國電信 App（App Store）：https://apps.apple.com/cn/app/513836029
- 小米 MIUI 11 AI 通話助理與澎湃 OS（IT 之家）：https://www.ithome.com/0/962/510.htm
- 騰訊雲 AI 語音代理（企業用）：https://www.tencentcloud.com/document/product/1229/70668
- 360 手機語音助理（2018，GizChina）：https://www.gizchina.com/360-mobiles/forthcoming-360-mobiles-smartphones-to-support-ai-voice-assistant
- SKT A. 通話功能開放全用戶（TelecomTV）：https://www.telecomtv.com/content/ai/skt-expands-a-s-call-features-to-all-subscribers-50058/
- SKT 新聞室（A. 電話 iPhone 錄音、摘要次數）：https://news.sktelecom.com/215045
- SKT AI 詐騙指示燈（SKT 英文新聞室）：https://news.sktelecom.com/en/582
- 台灣電信 AI 發展（公視）：https://news.pts.org.tw/article/798704
- 台灣電信 AI 發展（今周刊）：https://www.businesstoday.com.tw/article/category/183030/post/202512290044/
- Whoscall 語音示警與 iOS 26 整合（SOGI）：https://www.sogi.com.tw/articles/whoscall/6266633
- Whoscall 2025-09 改版（工商時報）：https://www.ctee.com.tw/news/20250914700020-430105

**電信轉接與號碼**
- 中華電信「指定轉接」（市話，官方）：https://www.cht.com.tw/home/consumer/broadband/service/valueadded/default-answer
- 轉接代碼整理（FlashGet，含 AT&T 代碼）：https://parental-control.flashget.com/zh-tw/how-to-forward-calls
- `**21` 轉接詐騙案例（民報）：https://www.peoplenews.tw/articles/anti-fraud-alert/30985
- 遠傳市話費率表（2026-07-22，官方 PDF）：https://enterprise.fetnet.net/content/dam/fetnet/user_resource/ebu/documents/市話服務費率方案表_20260722.pdf
- 條件轉接到 AI 助理的通用說明（IONOS）：https://www.ionos.com/help/ai/general-information/configuring-call-forwarding/
- Twilio 台灣號碼法規要求（官方）：https://www.twilio.com/en-us/guidelines/tw/regulatory
- Twilio 台灣語音價格（官方）：https://www.twilio.com/en-us/voice/pricing/tw
- Twilio 台灣語音規範（官方）：https://www.twilio.com/en-us/guidelines/tw/voice
- 070 網路電話制度（iThome）：https://www.ithome.com.tw/news/43148
- 是方電訊 070 服務（科技新報）：https://technews.tw/?p=442804

**語音 AI 技術與成本**
- OpenAI Realtime 價格整理（Fora Soft）：https://www.forasoft.com/article/openai-realtime-api-pricing
- GPT-Realtime-2 價格與延遲（Vantaige）：https://vantaige.io/blog/openai-gpt-realtime-2-voice-agent-setup-2026
- 即時語音 API 價格比較（日文）：https://aiagent-navi.com/special/realtime-api-pricing/
- 語音代理每分鐘成本（Inworld）：https://inworld.ai/resources/voice-agent-cost-per-minute-2026
- 語音代理平台成本比較（Speechify）：https://speechify.ai/blog/best-voice-agent-platforms-2026
- LiveKit 價格：https://www.livekit.com/pricing
- 語音直出 vs 三段串接（Gradium）：https://gradium.ai/content/cascaded-voice-agent-vs-speech-to-speech-2026
- 語音 AI 延遲（Coval）：https://www.coval.dev/blog/voice-ai-latency
- 聯發科 Breeze 3／台語模型（官方部落格）：https://www.mediatek.com/zh-tw/tek-talk-blogs/mediatek-research-breeze-3
- Breeze ASR 26、BreezyVoice 26（工商時報）：https://www.ctee.com.tw/news/20260304701327-430201

**平台限制與法律**
- Apple 開發者論壇：App 無法取得通話錄音：https://developer.apple.com/forums/thread/848299
- Google 禁止第三方通話錄音 App（Engadget）：https://www.engadget.com/google-is-banning-third-party-call-recording-apps-from-the-play-store-093201443.html
- 同上（SamMobile）：https://www.sammobile.com/news/android-call-recording-apps-wont-work-galaxy-phone-may-11/
- 台灣通話錄音法律（法律科普）：https://uptogo.com.tw/生活/對話可以錄音嗎/
- 全國法規資料庫（通保法、個資法原文請以此為準）：https://law.moj.gov.tw/

---

## 查不到或不確定的地方（彙整）

- 三家電信行動門號條件轉接的**官方計費方式**（最重要）、是否支援 `**004` 與自訂響鈴秒數、轉接時是否帶「原被叫號碼」。
- Truecaller Assistant 官方設定流程（官方說明頁抓不到，「用條件轉接」是業界通用做法的推論，Hiya 則有官方說明）。
- Twilio 台灣號碼的實際月租、接聽每分鐘費用、目前有沒有庫存；Vonage、Telnyx 台灣號碼。
- 070 業者現行批發價格與個人／新創申請條件。
- Hold for Me 最新國家清單；Samsung「AI 自己代接」在中文的支援度。
- 中國移動 AI 秘書、中國電信 AI 代接的價格；小米、360 目前是否有 AI 代接。
- SKT A. 電話目前的免費／付費摘要次數。
- 台灣國語、台語語音辨識的公開準確率數據（要自己實測）。
- 各語音 AI 每分鐘價格都是第三方整理，彼此不一致，要以官方價目為準。
- AI 代接錄音在台灣個資法下的告知義務（要問律師）。
