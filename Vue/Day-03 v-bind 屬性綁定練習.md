這次學習`v-bind`，`v-bind`是屬性綁定的指令。
`v-bind`是在做Vue專案中很常用的屬性之一，它可以用來綁定HTML元素的屬性，例如`src`、`href`、`title`等。
另外，它也可以用來做HTML元素在某狀態時的CSS樣式控制。

以下是這次的程式碼：

```javascript
<!-- 錯誤寫法！Vue 不支援在屬性內使用雙大括號 -->
<img src="{{ imagePath }}">

<!-- 正確寫法：使用 v-bind 綁定屬性 -->
<img v-bind:src="imagePath">

<!-- 最常用的簡寫法：直接寫一個冒號 `:` -->
<img :src="imagePath">
```

## 5 大常用應用情境
### 1. 綁定一般 HTML 屬性，控制圖片網址、超連結、表單的禁用（disabled）狀態等。
```javascript
<script setup>
import { ref } from 'vue'

const linkUrl = ref('https://vuejs.org')
const imgPath = ref('https://picsum.photos/200')
const isDisabled = ref(true)
</script>

<template>
  <a :href="linkUrl">前往 Vue 官網</a>
  <img :src="imgPath" alt="隨機圖片">
  
  <!-- 當 isDisabled 為 true 時，按鈕會加上 disabled 屬性 -->
  <button :disabled="isDisabled">無法點擊的按鈕</button>
</template>
```

### 2. 動態 Class 綁定，Vue 對 class 做了特別的優化，支援物件語法與陣列語法：
```javascript
<script setup>
import { ref } from 'vue'

const isActive = ref(true)
const hasError = ref(false)
const baseClass = ref('btn')
const sizeClass = ref('btn-large')
</script>

<template>
  <!-- 物件語法：{ 類別名稱: 布林值 } -->
  <div :class="{ active: isActive, 'text-danger': hasError }">
    動態 Class 區塊
  </div>

  <!-- 陣列語法：套用多個類別變數 -->
  <button :class="[baseClass, sizeClass]">按鈕</button>
</template>
```
### 3. 動態 Style 綁定 (行內樣式)
可以直接將 JS 物件綁定到 style 屬性上，注意 CSS 屬性名稱要改用駝峰式（camelCase）寫法：
```javascript
<script setup>
import { ref } from 'vue'

const textColor = ref('#3b82f6')
const fontSize = ref(20) // 單位會在 template 或物件中拼接
</script>

<template>
  <p :style="{ color: textColor, fontSize: fontSize + 'px' }">
    動態樣式文字
  </p>
</template>
```

### 4. 一次綁定多個屬性（物件展開）
如果你有一個包含多個屬性的物件，可以直接不帶參數使用 v-bind，一次將所有屬性套用到元素上：
```javascript
<script setup>
const buttonAttrs = {
  id: 'submit-btn',
  class: 'primary-button',
  disabled: false,
  'aria-label': '送出表單'
}
</script>

<template>
  <!-- 會自動展開變成 id="submit-btn" class="primary-button" ... -->
  <button v-bind="buttonAttrs">送出</button>
</template>
```

### 5. 組件間傳遞資料 (Pass Props)
當你開始使用自訂子組件時，v-bind 是將父組件資料傳給子組件的主要橋樑：
```javascript
<template>
  <!-- 把父組件的 user 變數傳給子組件的 user-info prop -->
  <UserProfile :user-info="user" />
</template>
```

## 實作練習

### (a) 練習程式碼

**:src 綁定**
```vue
<script setup>
import { ref } from 'vue'
const imgUrl = ref('https://picsum.photos/id/237/300/200')
</script>

<template>
  <img :src="imgUrl" alt="測試圖片">
</template>
```

**:class 物件語法**
```vue
<script setup>
import { ref } from 'vue'
const isBold = ref(false)

function toggleBold() {
  isBold.value = !isBold.value
}
</script>

<template>
  <p :class="{ 'text-red': isBold, 'font-bold': isBold }">這段文字會變色變粗</p>
  <button @click="toggleBold">切換樣式</button>
</template>

<style>
.text-red { color: red; }
.font-bold { font-weight: bold; }
</style>
```

**:style 物件語法**
```vue
<script setup>
import { ref } from 'vue'
const bgColor = ref('blue')

function toggleColor() {
  bgColor.value = bgColor.value === 'blue' ? 'green' : 'blue'
}
</script>

<template>
  <div :style="{ backgroundColor: bgColor, width: '100px', height: '100px' }"></div>
  <button @click="toggleColor">切換背景色</button>
</template>
```

### (b) 實際操作結果

> 跟程式碼一樣。

### (c) 卡關與解決方式

> 無。

### (d) 心得

> 了解了 v-bind 綁定屬性的用途，感覺以後會很常用到。

## 小撇步：Vue 3.4+ 同名簡寫法
在 Vue 3.4 及更新版本中，如果屬性名稱與變數名稱一模一樣，可以進一步簡寫：
```javascript
<script setup>
const id = ref('main-header')
</script>

<template>
  <!-- 舊寫法 -->
  <div :id="id"></div>

  <!-- Vue 3.4+ 同名簡寫：直接寫 :id 即可 -->
  <div :id></div>
</template>
```