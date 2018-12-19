# Lesson 2. Javascriptと非同期処理

## 目的

- 非同期処理の概念を理解する。
- コールバック関数、Promise、Async/Awaitそれぞれの特徴を理解する。

## 非同期処理とは

![非同期処理](./images/async-explained.png)

多くのプログラミング言語では、プログラムは上から下に順番に実行されていきます。しかし、例えばデータベースにアクセスしてデータを取得する場合など、処理に時間がかかるプログラムがその間に含まれる場合もあります。同期処理のプログラミング言語では、複数の処理を同時に行うマルチスレッドのモデルを取っています。JavaScriptシングルスレッドのモデルを取っており、同時に複数の処理を行うことが出来ません。

代わりにJavascriptでは、時間のかかる処理が完了するまでの間、他の処理を進めていき、その処理が終わった時に、またその処理の続きを再開する、ということが出来ます。これを非同期処理といいます。

また複数のイベント処理が山のように積み上がって(スタックといいます)おり、JavaScriptはこのスタック内をループしながら、時間のかかる処理が終わったかどうかをチェックしています。このループのことをイベントループと呼びます。

Javascriptで非同期処理を行うための仕組みはES5ではコールバック関数が使われており、ES6でPromise、ES7でAsync/Awaitという仕組みが導入されました。その3つそれぞれ使う機会が多いので、このレッスンではそれぞれを順番に解説していきます。

## コールバック関数

コールバック関数とは、ある関数(関数Aと呼ぶ)の引数として与えられる関数で、関数Aが一定の処理をしてから呼び出されることから、コールバック(呼び戻し)関数と呼ばれます。

例えば、以下がその例です。

```javascript
function logSomething(something) {
  console.log(something);
}

function callName(name, action) {
  let something = "Hello " + name;
  return action(name);
}

callName("World", logSomething);
```

実行結果: 
```
Hello World
```

この例ではcallNameという関数の2つ目の引数にlogSomethingという関数を与えています。このlogSomethingをコールバック関数と呼びます。またcallName関数のように別の関数を返す関数のことを高次関数(Higher-order Function)と呼びます。

## コールバック地獄

ES6以前のJavascriptでは非同期処理を書くにはコールバックを使うしかありませんでした。このコールバックは簡単な処理であればそれほど問題ないのですが、複数の非同期処理を連結して行う必要があるような場合、コールバック地獄と呼ばれる問題を引き起こしてしまいます。

例えば、以下ではfuncAという関数の結果を使ってfuncBを呼び出し、更にその結果を使ってfuncCを呼び出すfという関数を定義しています。

```javascript
function f() {
  return funcA(arg1, (err, res1) => {
    return funcB(res1, (err, res2) => {
      return funcC(res2, (err, res3) => {
        console.log(res3);
      }
    })
  })

}
```

上のように、関数の実行結果を次々と新しいコールバック関数に利用しているため、どんどんとネストが深くなってしまっています。上記の例は非常に単純化しており、エラー処理も入っていませんが、より複雑なプログラムだとどういう処理をしているのかが非常に把握しづらくなります。

## Promise

このコールバック地獄の問題を解決するために、ES6ではPromiseという新しい仕組みが導入されました。Promiseとは非同期処理の結果を保持するオブジェクトであり,またそのPromiseオブジェクトを利用した処理の仕組みのことを言います。

Promiseオブジェクトは以下の3つの状態を持っています。

1. pending: 初期の状態。処理実行前、あるいは実行中で処理が成功も失敗もしてない状態。
2. fullfilled: 処理が成功して完了した状態。
3. rejected: 処理が失敗した状態。

### Promiseオブジェクトの生成

Promiseオブジェクトは以下のようにして生成します。

```javascript
const promiseObj = new Promise((resolve, reject) => {
  // 何かの処理
  const result = doSomething();
  // 成功ならresolve、失敗ならrejectを返す。
  if (result === "Success") {
    // resolve状態のPromiseオブジェクトを返す。
    resolve(result);
  } else {
    // reject状態のPromiseオブジェクトを返す。
    reject(失敗の理由);
  }
});
```

### Promiseの基本的な使い方

#### then(onFullfillment, onRejection)

thenメソッドは、fullfilled状態のPromiseオブジェクトまたは、rejected状態のPromiseオブジェクト２つの引数を取ることが出来ます。thenメソッドはその処理の後に新しいPromiseオブジェクトを返します。

ここでは試しにパスワードの文字数のバリデーション(妥当性を調べること)をしてみましょう。パスワードの文字数が8文字以上なら成功で8文字未満ならエラーを返します。

```javascript
const validateCharCount = (password) => {
  return new Promise((resolve, reject) => {
    if (password.length >= 8) {
      resolve({
        password: password,
        status: "success"
      });
    } else {
      reject("パスワードは8文字以上で入力して下さい。");
    }
  })
}

let password = "testpassword"

validateCharCount(password).then(
  (result) => { // バリデーション成功時
    console.log("Valid Password");
  },
  (error) => { // バリデーション失敗時
    console.log(error); // => パスワードは8文字以上で入力して下さい。
  }
);
```

#### catch(onRejection)

catchメソッドは、rejected状態のオブジェクトを受け取って、処理を行うことが出来ます。catchメソッドの処理後は新しいPromiseオブジェクトを返します。実質的には、catchは`then(undefined, onRejection)`を呼び出しています。

例えば、ユーザーのプロフィール画像を画像サーバーから取得することを考えてみましょう。画像サーバーからの取得が失敗した場合は、デフォルトのプロフィール画像を別のサーバーから取得します。どちらも失敗したらエラーを返します。これは以下のようにして書くことが出来ます。

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

{
  getUserAvatar(1)
    .catch(getDefaultAvatar) // 画像取得に失敗したらgetDefaultAvatarを呼ぶ
    .then((img) => { // getUserAvatar、getDefaultAvatarいずれかが成功した場合
      document.body.appendChild(img);
    })
    .catch(() => { // getUserAvatar、getDefaultAvatarどちらも失敗
      console.log("Failed after all attempts.")
    })
    
}
```

この例では、catch、then、catchとメソッドを次々とつなぎ合わせています。これをPromiseチェーンと呼びます。このようにPromiseチェーンを活用することでCallback関数のようにネストが深くならずに複数の非同期処理を組み合わせることが出来ます。

**Promiseのサンプルコードはこちら:**
[js-unit02-lesson02-sample01](https://github.com/codegrit-jp-students/js-unit02-lesson02-sample01)

#### finally

finallyはPromiseチェーンの最後に必ず呼び出される処理を定義することが出来ます。例えば、処理を行っている間はロード画面を表示し、処理が完了したら成功、失敗問わずロード画面を非表示にする。というようなことが出来ます。


```javascript
{
  let isLoading = true; // ローディングを表示する。
  getUserAvatar(1)
    .catch(getDefaultAvatar) // 画像取得に失敗したらgetDefaultAvatarを呼ぶ
    .then((img) => { // getUserAvatar、getDefaultAvatarいずれかが成功した場合
      document.body.appendChild(img);
    })
    .catch(() => { // getUserAvatar、getDefaultAvatarどちらも失敗
      console.log("Failed after all attempts.")
    })
    .finally(() => {
      isLoading = false; // ローディングを非表示にする。
    })
}
```

### Promise.all

複数の処理を行い、全てが完了してから新しい処理を行いたいという場合にはPromise.allを使うと便利です。Promise.allはPromiseオブジェクトの配列を引数として取り、それぞれのPromise処理の結果の配列を値として取る新しいPromiseオブジェクトを返します。

```javascript
const promiseA = () => {
  return new Promise(resolve, reject) => {
    resolve(1)
  }
}

const promiseB = () => {
  return new Promise(resolve, reject) => {
    resolve("string")
  }
}

const promiseC = () => {
  return new Promise(resolve, reject) => {
    resolve(true)
  }
}

Promise.all([promiseA(), promiseB(), promiseC()].then((results) => {
  console.log(results) // => [1, "string", true]が返される
}))

```

例えば、ユーザー登録を行う時を考えてみましょう。この時、名前、メールアドレス、パスワードそれぞれのバリデーションを行い、全てが適切だった時だけ登録処理をしたいといます。この時次のように書くことが出来ます。

```javascript

const validateName = (name, errors = []) => {
  return new Promise((resolve, reject) => {
    let result = バリデーション処理
    if (result === "success") {
      resolve(name);
    } else {
      errors.push(new Error("名前の形式が異なります"))
      resolve();
    }
  });
}

const validateEmail = (email, errors = []) => {
  return new Promise((resolve, reject) => {
    let result = バリデーション処理
    if (result === "success") {
      resolve(email);
    } else {
      errors.push(new Error("パスワードの形式が異なります"));
      resolve();
    }
  });
}

cosnt validatePassword = (password, errors = []) => {
  return new Promise((resolve, reject) => {
    let result = バリデーション処理
    if (result === "success") {
      resolve(password)
    } else {
      errors.push(new Error("パスワードの形式が異なります"))
      resolve();
    }
  });
}

const validate = (name, email, password, errors = []) => {
  return Promise.all([
      validateName(name, errors), 
      validateEmail(email, errors), 
      validatePassword(password, errors)]
    .then((results) => {
      if errors.length === 0 {
        Promise.resolve { 
          valid: true
        }
      } else {
        Promise.reject(errors)
      }
    })
  )
}

let name = "code grit"
let email = "test@codegrit.jp"
let password = "testuser"
let errors = []

validate(name, email, password, errors)
.then(() => {
  createUser(name, email, password)
})
.catch((errs) => {
  let message = ""
  errs.forEach((error) => {
    message += error.message + " ";
  });
  console.log(message);
})

```

上記でvalidateName、validateEmail、validatePasswordの中で`reject`を利用していないことに注意してください。Promise.allでは、複数ある処理の内、どれから一つでもrejectedを返した場合、その時点で"rejected"状態のPromiseオブジェクトを返します。そのため、上記のバリデーションの例でrejectを利用すると、2つ以上問題のある入力がある場合でも、一つしかエラーが返ってきません。そのため、上記の例では`errors`という配列内にエラーオブジェクトを追加していくことで、複数のエラーメッセージを表示出来るようにしています。

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

## チャレンジ

[チャレンジ2](./challenge/README.md)

## 更に学ぼう

### 記事で学ぶ

- [Callback function - Mozilla(英語)](https://developer.mozilla.org/en-US/docs/Glossary/Callback_function)
- [Promise - Mozilla](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [async function - Mozilla](https://developer.mozilla.org/ja/docs/Web/JavaScript/Reference/Statements/async_function)
- [Promises - Google](https://developers.google.com/web/fundamentals/primers/promises?hl=ja)
- [Async functions - making promises friendly - Google](https://developers.google.com/web/fundamentals/primers/async-functions?hl=ja)