在本機裡試過了，創建了一個專案叫做"my-vlog"打算以後就拿這個做自己的部落客，
這次依照任務是過了`ref` `reactive` ，操作時感覺差不多，
`ref`在script操作時需要加上.value，`reactive`則不用，會出現差別是為底層邏輯設置就不同。
我們的接著往下看。

#### ref的底層邏輯
`ref`的底層邏輯是讓「基本型別（如數字、字串）」也能擁有像物件一樣的響應式能力（被攔截讀寫、收集依賴、觸發更新）。

我們直接上程式碼會比較好介紹：

```javascript

class RefImpl {
    // 這段的意思就是看看現在這個傳進來的值是什麼型別，是字串或數字，就直接拿來用，
    // 如果是物件，就透過 reactive() 轉為 Proxy
  constructor(value) {
    this._rawValue = toRaw(value);        
    // toRaw() 的作用是把 value 解出來，純粹是對照組，看看兩者是不是一樣，也就是原始值
    this._value = toReactive(value);      
    // toReactive() 把 value 變成 Proxy，如果 value 是物件，所以它就是用來判斷現在傳進的是不是物件，是物件就會用 reactive() 去轉，如果不是就直接拿來用。
  }

  // 當讀取 ref.value 時觸發
  get value() {
    trackRefValue(this);                  
    // 1. 收集依賴（Track），簡單來說就是「記錄誰在用我」。
    return this._value;
  }

  // 當修改 ref.value 時觸發
  set value(newVal) {
    if (hasChanged(newVal, this._rawValue)) {
      this._rawValue = toRaw(newVal);
      // 更新 rawValue，讓「下一次」setter 呼叫時 hasChanged 能拿最新值比對
      this._value = toReactive(newVal);   
      // 重新再判斷一次，是不是為物件，是就變Proxy
      triggerRefValue(this);              
      // 2. 觸發更新（Trigger），把原本那些有在用我的人叫起來說「喂！我改了，快更新！」
    }
  }
}

```

#### reactive的底層邏輯
它是專門為「物件和陣列」設計的，無法代理「基本型別（數字、字串、布林、null、undefined）」。
`ref`和`reactive`的目的是一樣的，都是要「記住誰在用我」跟「有改動就通知更新」，但攔截讀寫的方式不同：`reactive`是用 Proxy 包住整個物件去攔截；`ref`則是自己寫`.value`的 get/set 去攔截。只有當你塞給`ref`的是物件時，`ref`才會偷偷叫`reactive`來處理那個物件，如果塞的是數字、字串這種基本型別，就完全不會用到`reactive`。

```javascript
function reactive(target) {
  // 1. 只能代理物件或陣列
  if (typeof target !== 'object' || target === null) {
    return target;
  }

  // 2. 建立並回傳 Proxy 實例
  return new Proxy(target, {
    // 當有人讀取物件的屬性時（例如：state.count）
    get(target, key, receiver) {
      // 【步驟 A：依賴收集】記下是哪個畫面在看這個 key
      track(target, key); 
      
      // 使用 Reflect 安全地獲取原生物件的值
      const res = Reflect.get(target, key, receiver);
      
      // 💡 關鍵點：深層響應式
      // 如果讀出來的值「也是一個物件」，Vue 會在這裡「當場再用 reactive 包一次」
      if (typeof res === 'object' && res !== null) {
        return reactive(res);
      }
      
      return res;
    },

    // 當有人修改物件的屬性時（例如：state.count = 1）
    set(target, key, value, receiver) {
      const oldValue = target[key];
      // 使用 Reflect 修改原生物件的值
      const result = Reflect.set(target, key, value, receiver);
      
      // 【步驟 B：觸發更新】如果新舊值不同，就去通知那些記下的位置更新畫面
      if (oldValue !== value) {
        trigger(target, key);
      }
      
      return result; // 回傳 true 代表修改成功
    }
  });
}
```

現在Vue社群在使用上的建議是，盡量都用ref，不論是基本型別還是物件，都可以用ref來處理，因為ref會自動把物件轉為reactive，而且ref在操作上比較方便，只需要在get/set的時候加上.value即可。

那什麼時候用`reactive`呢？現在一般都只有表格再用了，例如：你只會改變裡面的幾個屬性，而且你確定整個物件不會被替換掉，就可以用`reactive`。