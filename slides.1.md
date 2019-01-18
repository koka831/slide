# Hello, World

***

## Before

```sh
python -c 'print("Hello, World")'
```

---

## After

<img src="./img/pyc.png" width="100%">

***

## who

<img src="./img/icon.jpg" width="200">
koka
GitHub: @koka831
Twitter: [@k_0ka](https://twitter.com/k_0ka)

---

<img src="./img/icon.jpg" width="100">

電気/通信専攻
普段はVue/React
趣味でRustでOS書いたりインタプリタ書いたり

***

## Table of Contents

プログラムがどのように実行されるのか  
CPythonを例に追っていく

***

## Start

```python
print("Hello, World")
```

---

- `print()`
- `"Hello, World"`

---

## いま無意識にやったこと

---

## lexer/字句解析
ソースコードを意味のあるまとまりの集合に変換する

---

(e.g.)
我輩は猫である。

→

```
["我輩", "は", "猫", "で", "ある", "。"]
```

***

# 次に

---

- `print()` → 関数
- `"Hello, World"` → 文字列

---

## プログラムでは
Parser/構文解析

---
(e.g.)

```
["我輩", "は", "猫", "で", "ある", "。"]
```
→

```
[
  ("我輩", 名詞),
  ("は", 接続詞),
  ("猫", 名詞),
  ("で", 助動詞),
  ("ある", 動詞),
  ("。", 終端記号)
]
```
---
この解析を左から行う→ LL/LR構文解析
意味のあるまとまりの端がどこか
- ここから新しい → 左端導出/LL
- ここまでがまとまり → 右端導出/LR

---

"は" って次にくる文字次第では  
名詞になることもあるんじゃない?

---

## 先読み

"は" の次は "猫"

→ 接続詞やな

---

- 左から読んでいき
- ひとつ先読みする
- 左端導出の構文解析

→

LL(1)

---

CPythonのParserはLL(1)  
[https://devguide.python.org/compiler/](https://devguide.python.org/compiler/#parse-trees)
---

## 意味のあるまとまり
Pythonでの意味のあるまとまりとは

***

## オブジェクト

Pythonでは関数も文字列もオブジェクト

---

# PyObject
全ての根源

[cpython/include/object.h](https://github.com/python/cpython/blob/master/Include/object.h#L109)

```c
typedef struct _object {
  // "CPython"のデバッグビルド時に利用
  _PyObject_HEAD_EXTRA
  // reference count
  Py_ssize_t ob_refcnt;
  // オブジェクトの型のインスタンスの参照
  struct _typeobject *ob_type;
} PyObject;
```

---

## 関数

[cpython/include/funcobject.h](https://github.com/python/cpython/blob/master/Include/funcobject.h#L21)

```c
typedef struct {
    PyObject_HEAD
    /* A code object, the __code__ attribute */
    PyObject *func_code;
    /* A dictionary (other mappings won't do) */
    PyObject *func_globals;
    PyObject *func_defaults;
    PyObject *func_kwdefaults;
    /* 中略 --- */

} PyFunctionObject;
```
---

# PyObject_HEAD
このオブジェクトが何なのかを示す情報
(e.g)

- int型のPyObject: PyInt_Type (という識別子)
- 関数のPyObject: PYFUNCTYPE

***

## 文字列
Pythonでの文字列の扱い 
"文字" の "配列"
---

## まず文字

[cpython/include/unicodeobject.h](https://github.com/python/cpython/blob/master/Include/cpython/unicodeobject.h#L239)

(unicodeの取扱がごちゃごちゃ書いてあるので省略)
---

## 配列

---

## PyVarObject
可変長のオブジェクト

[cpython/include/object.h](https://github.com/python/cpython/blob/master/Include/object.h#L118)

```c
typedef struct {  
    Py_ssize_t ob_refcnt;
    struct _typeobject *ob_type;
    Py_ssize_t ob_size;
} PyVarObject;
```
---

PyObjectとの違い:

```c
Py_ssize_t ob_size;
```
---

## なんで長さが必要?
プログラムはメモリに配置される  
Q.有限なメモリ上で
- 大きさが変わるオブジェクト
- 大きさが変わらないオブジェクト
どうすれば効率良く並べられる?

---

A.
大きさが変わらないオブジェクトは
詰めて並べて, 
大きさが変わるオブジェクトは
変わるもの同士まとめて並べる

---

```
+----------------------------------+
|不変| →              ← | 可変     |
+----------------------------------+
```

不変の部分: スタック領域

可変の部分: ヒープ領域

---

***

## ここまで

```c
PyObject(PYFUNCTYPE) {
  引数: PyVarObject(PySlice_Type("Hello, World"))
  func_code: // print()の実体
}
```

```
+----------------------------------+
|print()| →      ← |"Hello, World" |
+----------------------------------+

```

---

(今回の"Hello, World"のように
文字列が不変の場合には
スタックに配置されますが、
可変長のオブジェクトであることを強調)
---


