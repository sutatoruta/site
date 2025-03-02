# transform
* expected[meta header]
* function template[meta id-type]
* std[meta namespace]
* expected.void[meta class]
* cpp23[meta cpp]

```cpp
// expected<cv void, E>部分特殊化
template<class F> constexpr auto transform(F&& f) &;        // (1)
template<class F> constexpr auto transform(F&& f) const &;  // (2)
template<class F> constexpr auto transform(F&& f) &&;       // (3)
template<class F> constexpr auto transform(F&& f) const &&; // (4)
```

## 概要
正常値を保持していれば、`f`の呼び出し結果を`expected`の正常値として格納して返す。
エラー値を保持していれば、そのまま返す。

実際には複数オーバーロードが提供されるが、大まかには下記シグニチャのようにみなせる。
`transform`へは、引数をとらず`Return`型を返す関数や関数オブジェクトを与える。

```cpp
template <cv void, class E>
class expected {
  template <class Return>
  std::expected<Return, E> transform(function<Return()> func);
};
```
* function[link /reference/functional/function.md]


## テンプレートパラメータ制約
- (1), (2) : [`is_copy_constructible_v`](/reference/type_traits/is_copy_constructible.md)`<E> == true`
- (3), (4) : [`is_move_constructible_v`](/reference/type_traits/is_move_constructible.md)`<E> == true`


## 適格要件
- (1), (2) : 型`U`を[`remove_cvref_t`](/reference/type_traits/remove_cvref.md)`<`[`invoke_result_t`](/reference/type_traits/invoke_result.md)`<F>>`としたとき、次を全て満たすこと
    - `U`が`expected`の有効な正常値型である
    - `U`が（CV修飾された）`void`ではないとき、宣言`U u(`[`invoke`](/reference/functional/invoke.md)`(`[`std::forward`](/reference/utility/forward.md)`<F>(f)));`が妥当である
- (3), (4) : 型`U`を[`remove_cvref_t`](/reference/type_traits/remove_cvref.md)`<`[`invoke_result_t`](/reference/type_traits/invoke_result.md)`<F>>`としたとき、次を全て満たすこと
    - `U`が`expected`の有効な正常値型である
    - `U`が（CV修飾された）`void`ではないとき、宣言`U u(`[`invoke`](/reference/functional/invoke.md)`(`[`std::forward`](/reference/utility/forward.md)`<F>(f)));`が妥当である


## 効果
- (1), (2) : 次の効果をもつ
    - エラー値を保持していたら、`expected<U, E>(`[`unexpect`](../unexpect_t.md)`,` [`error()`](error.md)`)`を返す。
    - 型`U`が（CV修飾された）`void`でなければ、正常値を[`invoke`](/reference/functional/invoke.md)`(`[`std::forward`](/reference/utility/forward.md)`<F>(f))`で非直接リスト初期化した`expected<U, E>`オブジェクトを返す。
    - そうでなければ、[`invoke`](/reference/functional/invoke.md)`(`[`std::forward`](/reference/utility/forward.md)`<F>(f))`を評価し、`expected<U, E>()`を返す。
- (3), (4) : 次の効果をもつ
    - エラー値を保持していたら、`expected<U, E>(`[`unexpect`](../unexpect_t.md)`,` [`std::move`](/reference/utility/move.md)`(`[`error()`](error.md)`))`を返す。
    - 型`U`が（CV修飾された）`void`でなければ、正常値を[`invoke`](/reference/functional/invoke.md)`(`[`std::forward`](/reference/utility/forward.md)`<F>(f))`で非直接リスト初期化した`expected<U, E>`オブジェクトを返す。
    - そうでなければ、[`invoke`](/reference/functional/invoke.md)`(`[`std::forward`](/reference/utility/forward.md)`<F>(f))`を評価し、`expected<U, E>()`を返す。


## 備考
`transform`は、メソッドチェーンをサポートするモナド風(monadic)操作として導入された。


## 例
```cpp example
#include <cassert>
#include <expected>
#include <string>

int get_answer()
{
  return 42;
}

int main()
{
  std::expected<void, std::string> v1;
  assert(v1.transform(get_answer).value() == 42);

  std::expected<void, std::string> e1 = std::unexpected{"galaxy"};
  assert(e1.transform(get_answer).error() == "galaxy");
}
```
* transform[color ff0000]
* value()[link value.md]
* error()[link error.md]
* std::unexpected[link ../unexpected.md]

### 出力
```
```


## バージョン
### 言語
- C++23

### 処理系
- [Clang](/implementation.md#clang): ??
- [GCC](/implementation.md#gcc): 13.0
- [ICC](/implementation.md#icc): ??
- [Visual C++](/implementation.md#visual_cpp): ??


## 関連項目
- [`and_then()`](and_then.md)
- [`or_else()`](or_else.md)
- [`transform_error()`](transform_error.md)


## 参照
- [P2505R5 Monadic Functions for `std::expected`](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2022/p2505r5.html)
