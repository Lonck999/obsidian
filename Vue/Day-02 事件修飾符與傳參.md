這次要在專案裡面學`@click.prevent`、`@click.stop`，`@`是`v-on`的縮寫，`prevent`是阻止預設行為、`stop`是阻止事件冒泡。

這是這次準備的程式碼

```javascript

<script setup>
import { ref } from 'vue'
const num = ref(0);

const increment = () => {
	num.value++;
};

const showAlert = (message) => {
	alert(message);
};
</script>

<template>
    <div>{{ num }}</div>
	<!-- 練習一：點擊這個連結，數字會加 1，但網址不會跳轉 -->
	<a href="https://example.com" @click.prevent="increment"> 練習一：增加數字（阻止跳轉） </a>



	<!-- 練習二：點擊內層按鈕，只會觸發內層，不會觸發外層 -->

	<div @click="showAlert('觸發外層 div')">
		<button type="button" @click.stop="showAlert('觸發內層 button')">練習二：停止冒泡按鈕</button>
	</div>

	<!-- 練習三：同時使用兩種修飾符，點擊後觸發內層、不跳轉、不觸發外層 -->

	<div @click="showAlert('觸發外層 div')">
		<a href="https://example.com" @click.stop.prevent="increment"> 練習三：停止冒泡並阻止跳轉 </a>
	</div>

  <!-- 練習四：如果都不加的話會發生什麼事？ -->

  <div @click="showAlert('觸發外層 div')">
    <a href="https://example.com" @click="increment"> 練習四：不加修飾符 </a>
  </div>

  <div @click="showAlert('觸發外層 div')">
    <button type="button" @click="showAlert('觸發內層 button')">練習四：不加修飾符</button>
  </div>
</template>
```

## 事件修飾符

- `.prevent`：阻止元素的預設行為。像上面程式碼中的`<a>`連結，本來點下去會跳轉到`https://example.com`，加上`.prevent`後就不會跳轉了。
- `.stop`：阻止事件繼續往外層冒泡。上面程式碼中的內層`<a>`、`<button>`，點下去本來會讓事件一路往上冒泡，連帶觸發外層`<div>`的點擊事件；加上`.stop`後，事件就只會停在自己身上，不會再往外傳。

如果什麼修飾符都不加，就會像練習四那樣：點連結會跳轉，同時也會觸發外層 div；點 button 也會觸發外層 div，嚴重影響使用者操作的流程體驗。這種情況在電商網站很常見：商品卡片外層通常會包一個點擊事件（例如點卡片就導向商品頁），如果卡片裡面又有按鈕（例如加入購物車），沒加`.stop`的話，點按鈕會同時觸發卡片本身的點擊事件，導致同一次操作被重複記錄，讓收集到的使用者行為數據不準確。
