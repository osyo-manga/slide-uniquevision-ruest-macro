#### Rust LT会@オンライン
- - -

### Rust のマクロについて調べてみた

---

#### 自己紹介
- - -

* 名前：osyo
* Twitter : [@pink_bangbi](https://twitter.com/pink_bangbi)
* github  : [osyo-manga](https://github.com/osyo-manga)
* ブログ  : [Secret Garden(Instrumental)](http://secret-garden.hatenablog.com)
* 元 C++er の現 Rails エンジニア
* 趣味で Ruby にパッチを投げたりしてる
* Rust 歴は1日
    * 昔 struct + trait でコンパイル時処理で遊んだことはある
    * コンパイル時に階乗を行おうとしたりとか…
    * https://osyo-manga.github.io/slide-shinjukurb-58-rust/#/


---

### Rust のマクロについて調べてみた

---

#### マクロとは
- - -

* ソースコード中に繰り返し登場する特定の記述を、別の（短い）記述に置き換えることができる機能をマクロという。
* C / C++ ではプリプロセッサでプリプロセス時に機械的にコードを置換する機能

```cpp
#define PI 3.14
#define PLUS(a, b) a + b

auto f = PI + PI;       // auto f = 3.14 + 3.14
auto n = PLUS(1, 2) * 3 // auto n = 1 + 2 * 3
```

* Rust でも同等の機能があるが C / C++ よりもより強力な機能になっている
    * 単純な置換ではなくて構文自体を拡張できる
    * 構文木レベルでの拡張ができる（らしい
---

#### 簡単な使い方
- - -

* macro_rules! 名前 {} でマクロを定義する事ができる
* (引数) => { 展開後のコード } でマクロ本体を記述する

```rust
// マクロを定義する
// hello がマクロ名になる
macro_rules! hello {
	() => {
		println!("Hello!");
	};
}

fn main() {
	// 定義したマクロを呼び出す
	// hello!() のように ! を付けて関数っぽい呼び出しを行う
	hello!();  // println!("Hello!"); に置き換わる
	
	// () ではなくて [] や {} で呼び出すことができる
	hello![];
	// {} の場合は ; がなくてもいいらしい
	hello!{}
}
```

---

#### パターンを定義する
- - -

* (パターン) => { ... } で呼び出し元から渡された値を元に処理を分岐することができる

```rust
macro_rules! animal {
	// 引数の値で処理を切り分ける
	(cat) => { "にゃーん" };
	(dog) => { "わーん" };
	// 再帰的にマクロを呼び出すこともできる
	(cat and dog) => { animal!(cat).to_string() + " and " + animal!(dog) };
	(cat -> dog) => { animal!(cat).to_string() + " to " + animal!(dog) };
}

fn main() {
	// 引数の文字列によって呼び出されるマクロの処理が切り替わる
	println!("{}", animal!(cat));  // output: にゃーん
	println!("{}", animal!(dog));  // output: わーん
	println!("{}", animal!(cat and dog));  // output: にゃーん and わーん
	println!("{}", animal!(cat -> dog));   // output: にゃーん to わーん
}
```

---

#### パターンを変数で受け取る
- - -

* メタ変数と呼ばれるもので {変数名: フラグメント指定子} で値を受け取ることができる

```rust
macro_rules! plus {
	// $a と $b で引数の値を受け取る
	// expr は『式を受け取る』という意味
	// expr はフラグメント指定子と呼ばれる
	($a: expr, $b: expr) => { $a + $b };
}
macro_rules! calc {
	// 演算子も受け取る事ができる
	($a: expr, $op: tt, $b: expr) => { $a $op $b };
}

fn main() {
	println!("{}", plus!(1, 2));      // output: 3
	println!("{}", plus!(1, 2 * 3));  // output: 7
	println!("{}", calc!(1, -, 2));   // output: -1
}
```

---

#### デバッグ用のマクロをつくってみる
- - -

* 式とその結果の両方を出力するマクロを定義する

```rust
// 式を受け取ってその式の文字列と結果を表示する
macro_rules! debug {
	// stringify! で受け取った値を文字列に変換できる
	($expr: expr) => { println!("{} => {}", stringify!($expr), $expr) }
}

fn main() {
	let a = 42;
	debug!(1 + 2);  // output: 1 + 2 => 3
	debug!(a + 3);  // output: a + 3 => 45
	debug!(a.to_string() + "homu");  // a.to_string() + "homu" => 42homu
}
```

---

#### マクロのスコープ
- - -

* マクロ内のスコープは独立しているので変数は外から参照できない

```rust
macro_rules! test {
	// マクロ内のスコープは独立していて変数は外から参照できない
	() => {
		let a = 42;
		println!("{}", a);
	};
}

fn main() {
	test!();
	// error[E0425]: cannot find value `a` in this scope
	a;
}
```

---

#### マクロの優先順位(評価順)
- - -

* マクロが先に評価される

```cpp
#define test 1 + 2

// 1 + 2 * 3 と展開される
test * 3  // 7
```

```rust
macro_rules! test {
	() => {
		1 + 2
	};
}

fn main() {
	// (1 + 2) * 3 が評価される
	println!("{}", test!() * 3);  // output: 9
}
```

---

#### まとめ
- - -

* C / C++ と比べて Rust のマクロはかなり強力に見える
* この強力なマクロを Ruby で実装したい…
* まだまだおもしろい使い方があると思うので今後も調べていきたい
    * このあたりの話を普段から Rust を使っている人からいろいろと聞いてみたい
    * 実際のユースケースなどなど
* そもそも Rust のマクロの仕組みが（コンパイラや構文レベルで）わかってないのでそのあたりを今後は理解していきたい

---

#### 資料
- - -

* [Rustの全マクロ種別が分かったつもりになれる話! / rust-all-kinds-of-macro](https://speakerdeck.com/optim/rust-all-kinds-of-macro)
* [ためしておぼえる Rust のマクロ - Qiita](https://qiita.com/nirasan/items/cf05ac6d5a1ae17ae36f)
* [Rustのマクロを覚える - Qiita](https://qiita.com/k5n/items/758111b12740600cc58f)
* [マクロクラブ Rust支部 | κeenのHappy Hacκing Blog](https://keens.github.io/blog/2018/02/17/makurokurabu_rustshibu/)

---

### ご清聴
### ありがとうございました

