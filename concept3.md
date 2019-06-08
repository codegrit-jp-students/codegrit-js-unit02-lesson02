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

<iframe width="100%" height="300" src="//jsfiddle.net/codegrit_hiro/Lv3os6n8/embedded/js,html,css,result/dark/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>

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

const validatePassword = (password, errors = []) => {
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
      validatePassword(password, errors)])
    .then((results) => {
      if (errors.length === 0) {
        return Promise.resolve({
          valid: true
        });
      } else {
        return Promise.reject(errors)
      }
    })
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

<iframe width="100%" height="300" src="//jsfiddle.net/codegrit_hiro/cgoqL4wz/1/embedded/js,html,css,result/dark/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>

上記でvalidateName、validateEmail、validatePasswordの中で`reject`を利用していないことに注意してください。Promise.allでは、複数ある処理の内、どれから一つでもrejectedを返した場合、その時点で"rejected"状態のPromiseオブジェクトを返します。そのため、上記のバリデーションの例でrejectを利用すると、2つ以上問題のある入力がある場合でも、一つしかエラーが返ってきません。そのため、上記の例では`errors`という配列内にエラーオブジェクトを追加していくことで、複数のエラーメッセージを表示出来るようにしています。
