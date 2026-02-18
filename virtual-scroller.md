# Virtual Scroller（虛擬滾動）

## 什麼是 Virtual Scroller？

Virtual Scroller 是一種前端效能優化技術，用於高效渲染大量列表資料。其核心概念是：**只渲染使用者目前可見區域內的元素**，而非一次將所有項目都渲染到 DOM 中。

## 為什麼需要 Virtual Scroller？

當列表資料量龐大時（例如數千甚至數萬筆），若將所有項目都渲染為 DOM 節點，會導致：

- **記憶體佔用過高**：大量 DOM 節點消耗瀏覽器記憶體
- **渲染效能低落**：初次渲染與重繪時間大幅增加
- **滾動卡頓**：瀏覽器需要處理過多節點，導致 FPS 下降
- **頁面回應遲鈍**：使用者操作體驗明顯變差

## 運作原理

```
┌─────────────────────────┐
│      不可見區域（上方）    │  ← 以空白填充高度（padding / transform）
├─────────────────────────┤
│      Item 5             │
│      Item 6             │  ← 實際渲染的 DOM 節點（可視區域）
│      Item 7             │
│      Item 8             │
├─────────────────────────┤
│      不可見區域（下方）    │  ← 以空白填充高度（padding / transform）
└─────────────────────────┘
```

1. **計算可視範圍**：根據容器高度與滾動位置，計算出目前應顯示哪些項目
2. **只渲染可見項目**：僅將可視範圍內（加上少量 buffer）的項目渲染為真實 DOM
3. **模擬完整高度**：透過 padding 或 transform 撐出完整列表高度，讓捲軸行為正常
4. **動態替換內容**：當使用者滾動時，即時計算並替換需要顯示的項目

## 兩種常見模式

### 固定高度（Fixed Height）

- 每個項目高度一致
- 計算簡單，效能最佳
- 可直接透過 `scrollTop / itemHeight` 計算起始索引

### 動態高度（Dynamic Height）

- 每個項目高度不同
- 需要預先測量或估算每個項目高度
- 通常使用累加高度陣列或二分搜尋來定位項目

## 關鍵參數

| 參數 | 說明 |
|------|------|
| `itemCount` | 列表總項目數 |
| `itemHeight` | 每個項目的高度（固定模式）或估算高度 |
| `containerHeight` | 可視容器的高度 |
| `overscan` | 可視區域外額外渲染的項目數量（緩衝區），減少滾動時的白屏 |
| `scrollTop` | 目前的滾動偏移量 |

## 主流實作方案

| 套件 | 框架 | 特色 |
|------|------|------|
| `react-window` | React | 輕量、高效能，適合大多數場景 |
| `react-virtuoso` | React | 支援動態高度、群組、反向滾動 |
| `@tanstack/virtual` | 框架無關 | 支援 React / Vue / Solid 等多框架 |
| `vue-virtual-scroller` | Vue | Vue 生態系常用虛擬滾動方案 |
| `recyclerlistview` | React Native | 行動端虛擬列表解決方案 |

## 簡易實作概念（Vanilla JS）

```js
function virtualScroll({ container, items, itemHeight, renderItem }) {
  const visibleCount = Math.ceil(container.clientHeight / itemHeight);
  const totalHeight = items.length * itemHeight;

  // 建立滾動佔位元素
  const spacer = document.createElement('div');
  spacer.style.height = `${totalHeight}px`;
  container.appendChild(spacer);

  // 建立內容容器
  const content = document.createElement('div');
  content.style.position = 'relative';
  container.appendChild(content);

  container.addEventListener('scroll', () => {
    const scrollTop = container.scrollTop;
    const startIndex = Math.floor(scrollTop / itemHeight);
    const endIndex = Math.min(startIndex + visibleCount + 1, items.length);

    // 清除並重新渲染可見項目
    content.innerHTML = '';
    for (let i = startIndex; i < endIndex; i++) {
      const el = renderItem(items[i], i);
      el.style.position = 'absolute';
      el.style.top = `${i * itemHeight}px`;
      content.appendChild(el);
    }
  });
}
```

## 使用時的注意事項

- **設定合理的 overscan**：太少會出現白屏閃爍，太多則失去虛擬化的意義
- **避免在項目中使用複雜的 CSS 動畫**：可能影響滾動效能
- **動態高度場景需謹慎處理**：高度計算不準確會導致捲軸跳動
- **無障礙考量（Accessibility）**：虛擬化後螢幕閱讀器可能無法正確讀取所有項目
- **SEO 影響**：未渲染的項目不會被搜尋引擎爬取，需要注意是否影響 SEO 需求
