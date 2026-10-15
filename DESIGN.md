# 設計說明：偶像風格音樂實驗室

使用者選一位歌手的「風格模組」，用滑桿與開關即時調整一段原創循環音樂，聽到「如果用這位歌手的風格做一首歌，會是什麼聲音」。音樂由 [Strudel](https://strudel.cc) 在瀏覽器即時生成，不使用任何原曲音檔或原曲旋律。

← 回到 [README](README.md)

## 目標

- **目標使用者**：偶像粉絲、對音樂製作好奇但不懂樂理的人
- **核心體驗**：按下播放就有聲音；切換風格立刻聽出差異；拉動滑桿，聲音馬上跟著變
- **學習價值**：每個風格都附說明，讓使用者知道「為什麼聽起來像」——是速度、鼓的型態，還是和弦走向

## 功能

| 編號 | 功能 | 說明 | 狀態 |
| --- | --- | --- | --- |
| F1 | 風格模組切換 | 7 張歌手卡片，點選即切換音樂與配色 | 完成 |
| F2 | 播放／停止 | 瀏覽器規定音訊需由使用者點擊啟動 | 完成 |
| F3 | 音軌開關 | 鼓、貝斯、和弦、旋律、氛圍各自開關，聽出每一層的作用 | 完成 |
| F4 | 參數滑桿 | 速度（基準 BPM ±20）、能量（鼓的密度與力度）、亮度（濾波器）、空間感（殘響） | 完成 |
| F5 | 風格說明卡 | 速度、鼓組、和弦、音色，以及一句「為什麼聽起來像」 | 完成 |
| F6 | 程式碼面板 | 即時顯示產生的 Strudel 程式碼，可直接修改後執行 | 完成 |
| F7 | 混搭模式 | 各音軌分別選不同模組，例如 A 的鼓加 B 的和弦 | 未做 |
| F8 | 節拍視覺 | 背景隨節拍閃動 | 未做 |
| F9 | 分享設定 | 把目前模組與參數存成網址 | 未做 |
| F10 | 對照原曲 | 每位歌手 3 首參考歌曲都能在頁面內播放，用 A／B 切換比較生成版本與原曲，播放一邊時另一邊自動暫停 | 完成 |

## 風格模組

每個模組只描述「風格要素」，旋律與節奏都是依這些要素重新編寫的原創內容。

| 歌手 | 參考歌曲 | 類型 | 模組設定 | 代表色 |
| --- | --- | --- | --- | --- |
| 82MAJOR | Like Fire、Choke、Trophy | 偶像嘻哈 | 約 105 BPM；重大鼓、808 貝斯、合成器 stab；D 小調 | `#8C5A5A` |
| CHUNG HA | Play、México、Roller Coaster | 電子舞曲流行 | 約 120 BPM；四拍大鼓加熱帶打擊、跳八度貝斯；升 F 小調 | `#B88A9A` |
| TAEYEON | I、Why、Four Seasons | 抒情流行 × 雷鬼 | 87 BPM（半速）；雷鬼 one drop、反拍和弦、弦樂 pad；升 F 大調 | `#8A9BAE` |
| SUNMI | Heroine、Pporappippam、You Can't Sit With Us | 復古合成器流行 | 約 118 BPM；合成器 riff、空間感大的小鼓、突發打擊；A 小調 | `#8E7FA3` |
| CHANGMO | Maestro、Meteor、Hyperstar | 旋律嘻哈 × Trap | 約 140 BPM 半速感；trap hi-hat、長尾 808、鋼琴琶音；C 小調 | `#6F7F75` |
| ASH ISLAND | Lonely、Paranoid、Nightmare | 憂鬱旋律嘻哈 | 約 140 BPM 半速感；稀疏的鼓、撥弦吉他感琶音；E 小調 | `#5E6673` |
| B.I | Waterfall、Cosmos、Keep Me Up | 嘻哈 × 流行搖滾 | 約 100 BPM；流行搖滾鼓加過門、合唱感 pad；B 小調 | `#A08C6A` |

只有 TAEYEON〈Four Seasons〉的 BPM 與調性有公開資料（174 BPM，半速 87；升 F 大調），其他 BPM 為估計值。

### 每個模組的分析流程

1. 挑 3 首最能代表該歌手風格的歌
2. 查 BPM 與調性，取常見範圍
3. 聽鼓組型態、貝斯與音色，記下共同點
4. 寫成模組參數，再重新編寫原創的節奏與旋律
5. 試聽，請朋友盲測「聽起來像誰」，再微調

## 運作方式

```
使用者操作（選模組／拉滑桿／開關音軌）
        ↓
  更新 state 物件
        ↓
  buildCode()：依模組與參數組出一段 Strudel 程式碼
        ↓
  evaluate(code)：Strudel 解析並排程播放（不中斷節拍）
        ↓
  同步更新說明卡與程式碼面板
```

每個模組是一個物件，五條音軌各自是「參數 → Strudel 程式碼片段」的函式。例如亮度滑桿會換算成濾波器頻率，傳給和弦與旋律音軌：

```javascript
const lpf = Math.round(300 + Math.pow(state.bright, 2) * 7700);  // 300–8000 Hz
layers.push(m.chords(lpf));
```

新增一位歌手只要在 `MODULES` 陣列多加一個物件，並在 `asset/` 放入照片，不用改其他程式。

### 檔案結構

依作業規定，整個網頁寫在單一 `index.html`：

```
index.html
├── <style>     低飽和配色、銀色金屬點綴、手機版排版
├── <body>      版面
└── <script>
    ├── MODULES        7 個風格模組
    ├── buildCode()    產生 Strudel 程式碼
    ├── loadStrudel()  依序嘗試 unpkg、jsdelivr 載入 Strudel
    └── 介面事件與狀態
asset/          歌手照片
```

### 對照原曲

- 每位歌手的 3 首參考歌曲都嵌入 YouTube 影片，點選清單即可切換播放；以官方 MV 為主，沒有 MV 的歌曲改用官方表演影片、音樂節目舞台或音源
- 使用 [YouTube IFrame Player API](https://developers.google.com/youtube/iframe_api_reference) 偵測播放狀態，原曲開始播放時自動停止生成音樂，按下生成版本時自動暫停原曲
- 使用 `youtube-nocookie.com` 隱私加強模式嵌入
- API 載入失敗時退回一般嵌入；影片不允許嵌入時顯示原因並提供「改到 YouTube 觀看」連結
- YouTube 播放器需要網頁來源資訊，**用本機直接開檔（file://）時無法播放**，請使用 GitHub Pages 線上版本

### 載入失敗的處理

- unpkg 連不上時，自動改用 jsdelivr
- 鼓組音色來自 GitHub，若連不到則只用合成器音色播放，不會整個卡住
- 畫面上直接顯示失敗原因與解法

## 視覺設計

- **整體**：低飽和、時尚感，深灰底色加銀色金屬點綴
- **模組色**：選取模組時，頁面背景淡淡染上該歌手的代表色
- **照片**：平常為低飽和灰調，選中時恢復全彩
- **銀色元素**：播放鍵、滑桿控點、開關、選取卡片邊框使用銀色漸層，模擬金屬光澤
- **手機版**：歌手卡片改為可左右滑動的一列，讓控制項留在畫面上方

## 著作權原則

- **不使用**：原曲音檔、逐音轉錄的原曲旋律、歌詞
- **只借用**：BPM、調性、鼓組型態、和弦進行類型、音色類型等曲風共通要素
- 歌手照片版權屬原權利人，僅供課堂展示
- 原曲只透過 YouTube 官方頻道的影片嵌入播放，本站不存放任何原曲音檔
- 頁尾聲明本網頁為非官方課堂作業，與各歌手及經紀公司無關

## 參考資料

- [82MAJOR「Like Fire」回歸報導（Star News）](https://www.starnewskorea.com/en/music/2026/09/04/2026090414202187182)
- [Roller Coaster（Wikipedia）](https://en.wikipedia.org/wiki/Roller_Coaster_(Chung_Ha_song))
- [Roller Coaster 樂評（The Bias List）](https://thebiaslist.com/2018/01/18/song-review-chungha-roller-coaster/)
- [México 樂評（The Bias List）](https://thebiaslist.com/2026/09/01/song-review-chungha-mexico/)
- [Four Seasons（Wikipedia）](https://en.wikipedia.org/wiki/Four_Seasons_(song))
- [Four Seasons BPM 與調性（SongBPM）](https://songbpm.com/@taeyeon/four-seasons)
- [Heroine 樂評（The Bias List）](https://thebiaslist.com/2018/01/18/song-review-sunmi-heroine/)
- [SUNMI MV 造型（Vogue Singapore）](https://vogue.sg/sunmi-music-video-looks/)
- [Cosmos 樂評（The Bias List）](https://thebiaslist.com/2021/11/11/song-review-b-i-cosmos/)
- [Keep Me Up 樂評（The Bias List）](https://thebiaslist.com/2022/11/17/song-review-b-i-keep-me-up)
- [Waterfall 專輯樂評（NME）](https://www.nme.com/reviews/album/b-i-waterfall-illa-illa-review-2962587)
