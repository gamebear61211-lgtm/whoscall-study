# 來電辨識／防詐 App 基礎研究：資料、技術、法規（2026-10-23）

> 本文接續 `whoscall-research.md`（Whoscall 本身的競品資料不重複）。
> 標示說明：【已查證】＝有官方或可靠來源；【待確認】＝只有二手資料、論壇說法，或我查不到；【推論】＝我依資料推出來的判斷，不是查到的事實。
> 法律部分僅供規劃參考，正式上線前請找律師確認。

---

## 一、號碼資料從哪裡來、怎麼判斷詐騙

### 1-1 台灣政府開放資料（免費，授權為「政府資料開放授權條款第1版」，可商用但要標示出處）

| 資料集 | 內容 | 對我們有沒有用 |
|---|---|---|
| [165反詐騙諮詢專線_遭停止解析涉詐網站](https://data.gov.tw/dataset/176455) | 依《詐欺犯罪危害防制條例》第42條被停止解析的詐騙網址，2026-01 上架、持續更新 | **有用**：做「簡訊裡的網址」比對 |
| [165反詐騙諮詢專線_假投資(博弈)網站](https://data.gov.tw/dataset/160055) | 舊資料集，範圍已縮小、改看上面那份 | 參考用 |
| [165反詐騙諮詢專線－詐騙闢謠專區](https://data.gov.tw/dataset/38262) | 標題、發布時間、內文（文字公告），2026-10 仍有更新 | 可餵給 AI 當「最新詐騙手法」知識庫 |
| [165反詐騙諮詢專線-詐騙LINE ID](https://data.gov.tw/dataset/78432) | **已下架**：刑事局說新法上路後直接跟 LINE 通報處理 | 沒了 |
| [E.164 網路電話號碼核配現況](https://data.gov.tw/dataset/7172)（NCC） | 編碼字首、碼長、業者，CSV | **有用**：判斷是不是網路電話（070 等） |
| 行動通信網路業務用戶號碼核配現況（NCC，[鏡像站](https://scidm.nchc.org.tw/en/dataset/best_wish7171)） | 09xx 號碼區塊分給哪家業者 | 可判斷原始電信商（但有攜碼，不準）；鏡像版停在 2019，【待確認】最新版 |

**重點：台灣「沒有」可下載的詐騙電話號碼清單。**【已查證】165 只在 [165全民防騙網](https://165.npa.gov.tw/) 和 [165打詐儀錶板](https://165dashboard.tw) 提供「單筆查詢」，沒有開放批次下載或 API。Whoscall 的 165 資料是靠 2016 年跟刑事局簽 MOU 換來的（[TechBang 報導](https://www.techbang.com/posts/44989-fraud-trick-criminal-police-bureau-of-the-ministry-of-the-interior-in-cooperation-with-whoscall-signed-a-memorandum-of-anti-fraud)），報導說合作後只多了兩、三千筆號碼，可見 165 的電話資料量其實不大，真正的護城河是使用者回報。民間要怎麼申請 165 資料，我**查不到任何公開程序**【待確認】，只能直接去找刑事局談。
另外，用程式自動大量去 165 網站查詢（爬蟲）可能違反網站使用規範，不建議。

### 1-2 國際可付費的資料庫／API

| 服務 | 能給什麼 | 價格（查到的） | 條款重點／限制 |
|---|---|---|---|
| [Twilio Lookup v2](https://www.twilio.com/en-us/user-authentication-identity/pricing/lookup) | 號碼格式驗證（免費）、電信商＋號碼類型（手機／市話／網路電話）、來電者姓名 | 號碼類型 US$0.008/次；來電者姓名 US$0.01/次**但只限美國號碼**；身分比對 US$0.10/次 | 對台灣號碼主要只能拿到「號碼類型＋電信商」，**沒有詐騙分數、沒有台灣名稱** |
| [Numverify](https://numverify.com/signup)（apilayer） | 格式驗證、國家、電信商、號碼類型 | 免費 100 次/月；US$14.99/5,000 次；US$59.99/5 萬次；US$129.99/25 萬次 | 用完額度就停用；一樣**沒有詐騙判斷** |
| [IPQualityScore 電話信譽](https://ipqs.com/documentation/proxy-detection-api/phone-reputation) | 電信商、號碼類型、拋棄式號碼、近期有濫用紀錄的號碼 → 詐騙分數 | 有免費方案，額度【待確認】；大量資料產品報價從 US$499/月起（[Datarade](https://datarade.ai/data-products/ipqs-phone-number-validation-reputation-ipqualityscore-ipqs)） | 資料偏歐美網路註冊濫用，對台灣電話詐騙的覆蓋率【待確認】 |
| [Hiya](https://www.hiya.com/solutions/technology-partners) | 詐騙／騷擾判斷（供應 Samsung Smart Call、AT&T 等） | **沒有公開開發者價格**；公開的只有 Hiya Connect 品牌來電（約 US$99–835/月，[第三方整理](https://pricingsaas.com/companies/hiya)） | 只跟電信商、手機廠做 B2B 授權，新創要談很難【推論】 |
| Truecaller | 有給企業的 API（反查、垃圾分數），[但只開放給挑選過的企業](https://mynewsdesk.com/truecaller/pressreleases/truecaller-gears-up-to-release-their-api-861282) | 不公開 | 一般開發者能拿到的只有「用手機號碼登入」SDK（我的既有認知，【待確認】最新條款）；**沒有公開授權詐騙資料庫給第三方的管道** |
| 電信商 API（GSMA Open Gateway／CAMARA） | 號碼驗證、SIM 卡更換偵測等 | 依電信商 | [CAMARA 規格本身免費開源](https://camaraproject.org/wp-content/uploads/sites/12/2026/04/CAMARA-Onepager.pdf)，但**我查不到台灣中華／台哥大／遠傳有對外開放**【待確認】 |

**結論：國際 API 能幫你「把號碼分類」（手機/市話/網路電話/哪家電信），但沒有一家能便宜地給你「台灣這個號碼是詐騙」。台灣詐騙號碼資料只能自己累積。**

### 1-3 冷啟動：沒有使用者回報時怎麼辦【推論為主】

1. **白名單先行**：先做「這是正派號碼」比做「這是詐騙」容易又不會誤傷。來源：政府機關、銀行、醫院、物流公司公開的客服號碼（官網上都有）；Whoscall 早期也是用黃頁＋Google 地點資料起家。
2. **規則判斷**（不需要資料庫，第一天就能用）：見 1-4。
3. **把公開資料變成號碼包**：165 闢謠文、新聞報導裡出現的號碼，人工整理（量少但準）。
4. **找垂直族群當種子**：例如使用者本身是機場司機，司機群組之間互相回報「假叫車、假乘客」的號碼，一群人密集回報，資料品質會比泛用使用者高。
5. **企業認證號碼**：讓店家、車行主動登記自己的號碼（白名單），同時是收入來源。
6. **跟銀行／電信／165 合作**：參考 Whoscall 與兆豐、中信、遠傳的模式（見第三章）。

### 1-4 除了回報，還有哪些詐騙訊號

| 訊號 | 說明 | 來源 |
|---|---|---|
| **+開頭國際碼／偽冒 +886** | 台灣號碼從國外打進來，就是「境外偽冒」的典型。NCC 已要求電信業者在境外攔阻偽冒來話（報導說攔了逾 10 萬通），並規劃 +886 來電先播警示語音 | [1111 新聞](https://www.1111.com.tw/news/jobns/151277)、[iThome](https://www.ithome.com.tw/news/166016)；語音警示上線日期【待確認】 |
| **號碼格式異常** | 碼長不對、不存在的字首；網路電話字首（070）；刑事局與 Whoscall 合作時也特別研究「異常號碼格式、0800 來電」 | [NCC 核配資料](https://data.gov.tw/dataset/7172)、[TechBang](https://www.techbang.com/posts/44989-fraud-trick-criminal-police-bureau-of-the-ministry-of-the-interior-in-cooperation-with-whoscall-signed-a-memorandum-of-anti-fraud) |
| **電信網路驗證結果** | Android 10+ 的 `Call.Details` 有 `getCallerNumberVerificationStatus()`，失敗代表號碼可能是偽造的（美國 STIR/SHAKEN 機制；台灣電信商有沒有提供這個結果【待確認】，我猜多半是「未驗證」） | [Android 官方文件](https://developer.android.com/develop/connectivity/telecom/dialer-app/screen-calls) |
| **通話行為**（要靠很多使用者的匿名統計） | 短時間對大量不同人撥號、通話時間極短、多數人不接或掛斷、新號碼突然爆量、「鄰近號碼偽冒」（跟你前幾碼相同） | [TransNexus](https://transnexus.com/robocall-prevention/)、[Revenue.io](https://support.revenue.io/ringdna/telephony-101/caller-reputation-management/) |
| **AI 分析簡訊內容** | 抓「網址＋催促語氣＋要你點連結／加 LINE」等；網址可比對 165 停止解析清單。Google Messages 已用手機上的 Gemini Nano 做詐騙偵測，2026-10 又加了雲端版選項 | [BleepingComputer](https://www.bleepingcomputer.com/news/security/android-gets-ai-defenses-against-messaging-scams-fake-calls/)、[Droid Life 2026-10](https://www.droid-life.com/2026/10/14/google-messages-is-getting-a-massive-gemini-powered-scam-defense-upgrade/) |
| **AI 分析通話內容** | Google Pixel 在通話中即時判斷詐騙話術，全部在手機上處理。**但第三方 App 拿不到通話聲音**（見 2-5），這條路新進者幾乎走不通 | 同上 |

### 1-5 怎麼避免誤判（Whoscall 的已知破口：警察局電話曾被標成詐騙，[自由時報](https://news.ltn.com.tw/news/society/paper/1393089)）

- **分級顯示，不要一律「詐騙」**：區分「已確認詐騙（官方來源）」「多人回報可疑」「新號碼，資料不足」。學術專利也有提出給新號碼「中性」類別，避免新門號一打就被標（[USPTO 專利](https://image-ppubs.uspto.gov/dirsearch-public/print/downloadPdf/12542853)）。
- **回報要加權**：同一人大量回報、新帳號回報權重降低；要求多個不同使用者、跨時間回報才升級標籤。
- **白名單優先**：政府機關、銀行、醫院、認證企業的號碼不能被群眾標黑。
- **預設「警示」不要「直接封鎖」**：封鎖只對高信心號碼，或讓使用者自己選。
- **申訴管道**：號碼主人可以申訴下架標籤（也是個資法上的「更正權」，見 3-1）。
- **標籤會過期**：門號會被回收再分配，舊標籤要定期失效。

---

## 二、iOS 與 Android 怎麼攔電話、資料怎麼放在手機上

### 2-1 iOS：CallKit Call Directory extension（離線號碼包）【已查證】

- 運作方式：App 先把「號碼＋標籤」或「要封鎖的號碼」整包交給系統，來電時系統自己比對。**來電當下 App 不會被叫醒，也不能即時上網查**（[Apple 文件](https://developer.apple.com/documentation/callkit/identifying-and-blocking-calls.md)）。
- 比對順序：系統先查使用者通訊錄 → 再查你的號碼包。號碼要「由小到大排序」後加入。
- **數量上限**：Apple **沒有公布官方上限**，只有一個「條目太多」錯誤。開發者論壇實測約 100 萬～200 萬筆，主要卡在 extension 的記憶體限制；有人說競品能放超過 1,000 萬筆（[論壇 1](https://developer.apple.com/forums/thread/769594)、[論壇 2](https://developer.apple.com/forums/thread/681793)）【待確認，需實機測】。
- **更新方式**：主 App 呼叫 `CXCallDirectoryManager.reloadExtension` 讓系統重新載入；支援「增量更新」（只加減變動的號碼）。App 在背景時不一定能準時更新，所以號碼包會有時差。
- 使用者要自己到「設定 → 電話 → 來電封鎖與識別」打開你的 App，這一步很多人不會做，要在 App 內教學。

### 2-2 iOS 18+：Live Caller ID Lookup（即時雲端查詢）【已查證，但上線難度高】

- 來電時系統直接問你的伺服器「這號碼是誰、要不要擋」，畫面上顯示名稱並記在通話紀錄（[Apple 隱私說明](https://developer.apple.com/tutorials/data/documentation/identitylookup/understanding-how-live-caller-id-lookup-preserves-privacy.md)）。iOS 18.0 以上（[API 文件](https://developer.apple.com/documentation/identitylookup/livecalleridlookupextensioncontext.md)）。
- **隱私要求（你必須做到的）**：
  1. 透過 Apple 的 Oblivious HTTP 雙層轉送：Apple 一層、第三方一層，你的伺服器看不到使用者 IP。
  2. Privacy Pass 匿名驗證：你知道「是付費會員」但不知道「是哪個會員在查」。
  3. **PIR（私密資訊檢索）**：你的伺服器要跑一個特殊的加密資料庫，查詢時連「被查的號碼」你都看不到。Apple 提供 [範例程式](https://github.com/apple/live-caller-id-lookup-example)，但另一個 [pir-service-example 明講不能直接上正式環境](https://swiftpackageregistry.com/apple/pir-service-example)。
- **申請流程**：要在 CloudKit Console 的 Identity & Trust 頁面登記並向 Apple 申請。論壇大量抱怨卡審查 3 週到 4 個月、TestFlight 能用但正式版失敗、iOS 18 初期成功率很低（[論壇](https://developer.apple.com/forums/thread/818336)、[論壇](https://developer.apple.com/forums/thread/817921)、[論壇](https://developer.apple.com/forums/thread/760693)）。
- **對新進者的意義**：技術門檻高（要懂同態加密的伺服器）＋審核不透明，**MVP 不建議做**，先用 Call Directory 離線包。好處是天然符合「不知道你接了誰的電話」的隱私賣點。

### 2-3 iOS 26 內建 Call Screening【部分查證】

- 系統自己幫你接陌生來電，請對方報姓名和來意，畫面即時顯示逐字稿；支援中文（Mandarin）等語言，不需要 Apple Intelligence（[Tom's Guide](https://tomsguide.com/phones/iphones/ios-26-call-screening-promises-to-keep-spam-callers-at-bay-heres-how-to-set-it-up)）。2025-09-15 正式推出。
- 辨識來源包含 Apple Business Connect、電信商，以及使用者安裝的第三方辨識 App（[MacSales](https://eshop.macsales.com/blog/97828-silence-of-the-spams-how-to-enable-ios-26-iphone-call-screening-filters/)）。
- **我查不到任何給開發者的 Call Screening API**【待確認】：第三方 App 不能自己做「代接」，也拿不到這段逐字稿。
- **威脅【推論】**：Apple 免費內建「AI 代接陌生來電」，等於直接吃掉「AI 秘書代接」的一大塊需求；Call Directory 標記的號碼在篩選時算不算「已知」，我查不到官方說法。

### 2-4 iOS 簡訊過濾 extension（IdentityLookup / `ILMessageFilterExtension`）【已查證】

- 只處理「陌生人」傳來的簡訊（通訊錄裡的人、你回覆過 3 次的人不會進來）；**同時間只能啟用一個過濾 App**（[WWDC22](https://developer.apple.com/videos/play/wwdc2022/110341/)、[Apple 隱私頁](https://apple.com/legal/privacy/data/en/sms-filtering)）。
- 可把簡訊分到「交易、推廣、垃圾」與最多 5 個子分類。
- 在手機上判斷不了時，可以把「寄件人＋內容」送到你的伺服器問（[Apple 文件](https://developer.apple.com/documentation/identitylookup/sms-and-mms-message-filtering.md)）——**這就是可以接 AI 判斷詐騙簡訊的地方**，但會碰到個資問題（見 3-1）。

### 2-5 Android【已查證】

- **CallScreeningService**（Android 10 以上）：來電響鈴前系統先問你的 App，可以：封鎖、拒接、靜音、不記入通話紀錄、不跳未接通知；只處理「不在通訊錄裡」的號碼。**不需要讀通話紀錄權限**（[Android 文件](https://developer.android.com/develop/connectivity/telecom/dialer-app/screen-calls)）。
- **要先取得「來電過濾（Call screening）」預設角色**：用 `RoleManager.ROLE_CALL_SCREENING` 請使用者同意。App 要在約 5 秒內回應，不然電話照響（AOSP 原始碼）。
- 跟 iOS 最大的差別：**可以當下即時上網查詢**、也可以疊一個畫面或通知顯示來電者資訊（Whoscall 在 Android 功能較強就是這原因）。
- **Google Play 對通話紀錄／簡訊權限的政策**（[Play 政策](https://support.google.com/googleplay/android-developer/answer/10208820)）：
  - 原則上要成為「預設簡訊／電話／助理 App」才能申請 READ_SMS、READ_CALL_LOG 等權限。
  - 例外清單裡**明確有「來電辨識、垃圾電話偵測與封鎖」**和「防簡訊釣魚（需有顯著保護使用者的紀錄）」，但都要經過 Google 審核，屬「暫時例外」。
  - 新 App 建議：電話只用 CallScreeningService（免權限）；簡訊先不要讀，改成「使用者自己轉貼/分享可疑簡訊給 App 判斷」，避開審核風險【推論】。
- **通話錄音**：Google Play 2022-05 起禁止第三方 App 用無障礙 API 錄通話，只有預載的系統撥號 App 能錄（[The Register](https://www.theregister.com/2022/04/22/google_banning_thirdparty_callrecording_apps/)）。iOS 本來就不給第三方錄通話。→ **第三方 App 在兩個平台都拿不到電信通話的聲音**。
- 那市面上的「AI 代接」怎麼做？用電信商的「無人接聽／忙線轉接」（如 `**62`、`**67`）把電話轉到雲端號碼，再由雲端 AI 接（[Call.io](https://apps.apple.com/app/id6754271278)、[Fonio 設定說明](https://fonio-knowledge-base.help.usepylon.com/articles/7960779505-rufweiterleitung-einrichten)）。台灣電信商轉接是否另計費、要用哪種雲端號碼（需要 NCC 執照的業者提供）【待確認】。

### 2-6 Flutter 或 React Native 能做到什麼程度【部分查證＋推論】

- **主畫面、帳號、付費、回報流程、AI 聊天助理**：Flutter／React Native 都沒問題。
- **攔截核心一定要寫原生程式**：iOS 的 Call Directory、Live Caller ID、簡訊過濾都是「extension」，是獨立的小程式，只能用 Swift 寫，跟 Flutter 主程式透過共用資料夾（App Group）交換號碼包；Android 的 CallScreeningService 要用 Kotlin 寫。
- 現成套件很少：Flutter 只有 Voximplant 做過 Call Directory 串接，而且他們自己說要在 Xcode 手動加 extension、自己做號碼儲存（[DEV 文章](https://dev.to/imaximova/how-we-make-flutter-work-with-callkit-call-directory-5334)、[pub.dev](https://pub.dev/packages/flutter_callkit_voximplant/versions/2.3.0)）；Android 端、React Native 端我沒找到成熟套件。
- 建議【推論】：選 Flutter（一套 UI 兩平台），外加兩個原生小模組（Swift extension ＋ Kotlin service），各約數百行程式；這部分可以讓 AI 寫，但一定要實機測試。

---

## 三、台灣法規與商業模式

### 3-1 個人資料保護法

- **電話號碼＋被標的名稱/標籤＝個資**（能識別特定自然人時）。來電辨識 App 天生在處理「第三人（打電話的人）」的個資，這是本行最大的法律風險【推論】。
  - Apple 審查也會擋：有來電辨識 App 被以 5.1.1「從公開來源蒐集資料建立個人檔案」退件，反查他人號碼的 App 被以 5.1.2 退件（[論壇 1](https://developer.apple.com/forums/thread/801880)、[論壇 2](https://developer.apple.com/forums/thread/128727)）。
  - 做法【推論】：只標「類型」（詐騙、推銷、外送、客服）不標個人姓名；個人號碼不提供公開反查；**不上傳通訊錄**（Whoscall 的已知弱點，也是我們的差異化）；提供號碼主人申訴更正、刪除。
- **2025-11-11 公布個資法修正**，重點：成立個人資料保護委員會；發生外洩須在知悉 72 小時內通知當事人，達一定規模要通報；**施行日期還沒定**（2026-03 律師文章仍說待定）（[理律](https://www.leeandli.com/TW/NewslettersDetail/7584.htm)、[中央社](https://www.cna.com.tw/news/aipl/202506160188.aspx)）。草案階段報導罰鍰最高 1,000 萬元，最終數字【待確認】。
- 簡訊內容送雲端 AI 分析：要在隱私權政策清楚告知、取得同意，能在手機端判斷就不要上傳【推論】。

### 3-2 通訊保障及監察法（通話錄音、AI 代接）

- 第 24 條：違法監察他人通訊處 5 年以下徒刑。第 29 條第 3 款：**「監察者為通訊之一方或已得通訊之一方事先同意，而非出於不法目的」不罰**＝台灣實務的「單方同意」原則（[法律百科](https://www.legis-pedia.com/QA/question/930)）。所以使用者自己錄自己的電話原則上合法。
- **AI 代接**【推論，查不到直接判例或函釋】：AI 是代表使用者接聽，使用者事先同意，可以主張落在「通訊一方同意」；但錄音與逐字稿存在你的伺服器，等於你公司持有「打電話那個人」的聲音與談話內容，個資法責任在你。
- **告知義務**：通保法本身沒有要求告知對方「正在錄音」。2025-12-23 三讀的《人工智慧基本法》是框架法，有「透明、揭露」原則與高風險 AI 警語，但**沒有直接罰則，也沒有規定 AI 打電話一定要說自己是 AI**；細節授權數發部另訂（[科技新報](https://technews.tw/2026/01/07/ai-basic-law-tw/)、[聯合報](https://udn.com/news/story/7238/9223381)）。
- 建議：AI 一接起來就說「您好，我是○○的 AI 助理，本通話會被記錄以便轉達」——成本幾乎為零，可降低個資與未來法規風險，也是 iOS 26 Call Screening 的做法【推論】。

### 3-3 NCC 與電信相關規定

- 《詐欺犯罪危害防制條例》課以**電信業者**四大防詐義務（實名、擋可疑用戶、防門號轉讓、控管高風險用戶），以及商業簡訊內含網址／電話的檢查（[理律](https://www.leeandli.com/TW/NewslettersDetail/7290.htm)、[iThome](https://www.ithome.com.tw/news/162783)）。這些是對電信商的義務，**單純的來電辨識 App 不是電信事業，不用申請 NCC 執照**【推論，我查不到 NCC 對此類 App 的直接規範】。
- **會碰到 NCC 的情況**：如果你自己提供電話號碼、語音轉接、網路電話服務（例如 AI 代接要給使用者一個轉接號碼），就可能構成電信服務；NCC 曾因業者轉接未登記電信事業的語音服務裁罰 10 萬元（[台南市公告](https://culture.tainan.gov.tw/df_ufiles/215/三通網轉接未登記為電信事業者之語音服務，違反電信管理法第13條第5項，處罰鍰新臺幣10萬元.pdf)）。→ AI 代接的號碼要跟有執照的業者合作，不要自己架。

### 3-4 App 上架審查重點

- **Apple**：資料最少化（5.1.1）、不能蒐集第三人資料建立檔案（5.1.2）；Live Caller ID 另外要 Apple 核准；iOS 沒有通話紀錄 API，不能讀。
- **Google Play**：讀簡訊／通話紀錄要預設角色或申請例外（見 2-5）；不能錄通話；隱私權政策與資料安全表單要寫清楚。
- 兩邊都要準備：隱私權政策中文版、刪除帳號功能、資料用途說明。

### 3-5 收費方式與合作對象

| 模式 | 案例／依據 | 對新進者的可行性【推論】 |
|---|---|---|
| **訂閱（App 內購）** | Whoscall 月費 NT$95／年費 NT$890 | 主力，但要給比 Whoscall 更明確的理由（零廣告、隱私、AI 助理） |
| **電信帳單代收** | 遠傳獨家在電信帳單賣 Whoscall 進階版，試行後訂閱數翻倍（[聯合報](https://udn.com/news/story/7240/9203491)）；中華電信曾推市話版（[KOC](https://koc.com.tw/?p=298044)，可能已停） | 電信商偏好跟有規模的品牌合作，新進者難；可以等有用戶量再談 |
| **銀行合作** | 兆豐：代表號用 Whoscall 認證、送 10 萬組兩個月兌換碼（[iThome](https://www.ithome.com.tw/pr/167629)、[TechBang](https://www.techbang.com/posts/128600-mega-bank-anti-fraud-whoscall-free)）；中信：送 65 歲以上客戶 4,000 組 | 銀行有「防詐 KPI」壓力且願意買單；切「長輩防詐」或「家人共享」很對味 |
| **企業認證號碼（白名單收費）** | Whoscall 認證商家號碼 | 可從垂直產業（車行、物流、診所）開始，收入也幫忙冷啟動 |
| **165／刑事局** | 2016 MOU 換資料；2023 刑事局送 10 萬組 Whoscall 序號（[Cool3c](https://www.cool3c.com/article/203644)） | 官方合作要有用戶規模與公信力，屬中長期目標；民間申請管道我查不到 |
| **廣告** | Whoscall 營收 36% | 跟「零廣告」定位衝突，不建議 |

---

## 對新進者的關鍵結論

1. 台灣沒有可下載的詐騙電話清單，165 只能單筆查；國際 API（Twilio、Numverify）只能分號碼類型，不知道台灣號碼是不是詐騙，詐騙資料只能自己累積。
2. 冷啟動先做「白名單＋規則判斷」（+886 境外偽冒、網路電話字首、異常格式、165 停止解析網址），再找一個垂直族群（例如司機圈）密集回報。
3. 誤判是 Whoscall 的弱點：分級顯示、回報加權、官方白名單優先、可申訴、標籤會過期，做成我們的賣點。
4. iOS 只能先放離線號碼包（Call Directory，實測約 100–200 萬筆，Apple 沒公布上限）；Live Caller ID 要架 PIR 加密伺服器又要 Apple 審核，等之後再做。
5. Android 用 CallScreeningService 免敏感權限就能即時查、即時擋；讀簡訊／通話紀錄要過 Google 審核，先避開。
6. 第三方 App 兩個平台都拿不到通話聲音；AI 代接只能走「電信轉接到雲端號碼」，而且 iOS 26 已經免費內建 AI 代接陌生來電，這塊要重新想差異化。
7. Flutter 做主 App 可以，但攔截核心一定要寫原生 Swift／Kotlin 小模組。
8. 法律上最大風險是處理「打電話那個人」的個資：不標個人姓名、不上傳通訊錄、不公開反查；AI 代接開頭主動說「我是 AI、會記錄」。
9. 單純來電辨識不用 NCC 執照，但自己提供轉接號碼／語音服務就會碰電信法規，要找有執照的業者合作。
10. 收費以訂閱為主，最務實的 B2B 突破口是銀行（長輩防詐）和企業認證號碼；電信代收和 165 合作要等有用戶規模再談。
