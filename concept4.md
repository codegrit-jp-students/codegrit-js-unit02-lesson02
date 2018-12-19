## Async/Await

ES7では非同期処理を同期処理的に書く方法としてAsync/Awaitが導入されました。AsyncファンクションはPromiseオブジェクトを返します。そのためPromiseと同様にthenを利用して繋げていくことが出来ます。

### Asyncファンクションの書き方

```javascript
const asyncAction = async () => {
  const result = await doSomething(); 
  // doSomethingという非同期処理に対してawaitを追加することで処理完了を待つ。
  if (result === "成功") {
    // 結果に基づいた処理
  } else {
    // 結果に基づいた処理
  }
}

async function ファンクション名() {
  const result = await doSomething(); 
  // doSomethingという非同期処理に対してawaitを追加することで処理完了を待つ。
  if (result === "成功") {
    // 結果に基づいた処理
  } else {
    // 結果に基づいた処理
  }
}
```


### Asyncファンクションを使う

例えば、先ほどのバリデーションの処理をasync/awaitを書くと以下のようになります。

```javascript
const validateName = async (name) => {
  let result = await バリデーション処理
  if (result === "success") {
    return {valid: true};
  } else {
    return new Error("名前の形式が異なります。");
  }
}
```

### try/catchを使う

Promiseの場合はreject時の場合の処理を`promiseA.catch(onRejcet)`のように書きました。Async/Awaitを使う場合にはtry...catch文を利用して以下のように書きます。

```javascript
async asyncA() {
  try {
    let result = await promiseA
  } catch {
    // reject時の処理
  }
}
```

Promise.catchの説明時に利用した、画像読み込みの例をAsync/Awaitで書き直すと以下のようになります。

```javascript
const getUserAvatar = (userId) => {
  return new Promise((resolve, reject) => {
    let img = new Image();
    img.onload = () => {
      resolve(img);
    }
    img.onerror = () => {
      reject("Failed")
    }
    img.src = `../images/${userId}.png`;
  });
};

const getDefaultAvatar = () => {
  return new Promise((resolve, reject) => {
    let img = new Image();
    img.onload = () => {
      resolve(img);
    }
    img.onerror = () => {
      reject("Failed")
    }
    img.src = "../images/default.png";
  });
};

const getAvatar = async (userId) => {
  let image;
  try {
    image = await getUserAvatar(userId);
  } catch (e) {
    image = await getDefaultAvatar();
  }
  return image
}

const showAvatar = async (userId) => {
  let result;
  try {
    image = await getAvatar(userId);
    document.body.appendChild(image);
  } catch (e) {
    console.log(e);
  }
}

{
  showAvatar(2); // => デフォルトのプロフィールイメージが表示される。
}
```
**Async/Awaitのサンプルコードはこちら:**
[js-unit02-lesson02-sample02](https://github.com/codegrit-jp-students/js-unit02-lesson02-sample02)

## 更に学ぼう

### 記事で学ぶ

- [Callback function - Mozilla(英語)](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)
- [Promise - Mozilla](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [async function - Mozilla](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/async_function)
- [Promises - Google](https://developers.google.com/web/fundamentals/primers/promises?hl=ja)
- [Async functions - making promises friendly - Google](https://developers.google.com/web/fundamentals/primers/async-functions?hl=ja)