---
layout: post
title: "Programming Tips: 参照渡しと参照型の値渡し"
date: 2025-01-06
---

# はじめに

さて、いきなり問題であるが、次の python スクリプトを実行した時、
何と表示されるか、すぐに答えられるであろうか。

```python
#!/usr/bin/env  python3

def func(x):
    # print(id(x))
    x.append(3)
    x = [1, 2, 3, 4]
    x.append(5)
    # print(id(x))
    print(f"@func.x = {x}")

a = [1, 2]
func(a)
print(f"@main.a = {a}")
```

まず func の中にある print は、誰も間違えることはないだろう。
```
@func.x = [1, 2, 3, 4, 5]
```
である。

問題は、最終行の print(f"@main.a = {a}") である。
つまり func(a) の呼び出しで a がどのように変化するかである。

ここでよくある勘違いは python は参照渡し
だから func の引数 x と a は同じ変数。よって

```
@main.a = [1, 2, 3, 4, 5]
```

...

というのは、大きな間違いである。
ちなみに、先に正解をいうと
```
@main.a = [1, 2, 3]
```

である。実は python の関数呼び出しは参照渡しではない。


#   2.  C++ の場合

先ほどの python とほぼ同等の動作を C++ で書くと
以下のようになる。

なお C++ では参照が使えるのだが、
参照は一度アドレスをセットすると、別のアドレスに書き換える事ができない。
このため、先の処理と同等の内容を書くために、
参照ではなくポインタを使っている。

```
#if 0
g++  -o test -g -O0 -std=c++11 test.cpp
exit 0
#endif

#include <iostream>
#include <list>

void show_list(const std::list<int> * const ls)
{
    const std::list<int>::const_iterator itrEnd = ls->end();
    for (std::list<int>::const_iterator
            itr = ls->begin(); itr != itrEnd; ++ itr )
    {
        fprintf(stdout, "%d, ", *itr);
    }
    fprintf(stdout, "ls = %p\n", ls);
    return;
}

void func(std::list<int> *x)
{
    fprintf(stderr, "%p\n", x);
    x->push_back(3);
    x = new std::list<int>( {1, 2, 3, 4} );
    fprintf(stderr, "%p\n", x);
    x->push_back(5);
    show_list(x);
}

int main(int argc, char * argv[])
{
    std::list<int>   a = { 1, 2 };
    std::list<int> * pa = &a;

    show_list(pa);      //  1, 2
    func(pa);
    show_list(pa);      //  1,2,3,4,5 ? => 正解は 1,2,3,
}
```

func の動作であるが、問題になるのは、
```
    x = new std::list<int>( {1, 2, 3, 4} );
```

の行である。ここで新しいインスタンスを生成して x に代入している。
ポインタとは、メモリ上の番地を示すだけで、
その実体は大抵 0x1234 のような数値である。

そして、ポインタを引数に渡す場合、
そのポインタ（上記の 0x1234 のような番地を示す数値）自体は
値渡しされることに注意が必要である。

よって、最初の
x->push_back(3) までは、x は main の pa と同じ番地を指しているから、
ここで 3 を末尾に追加した事は main 側にも影響を与える。
この時点でリストの中身は {1, 2, 3} になっている。

しかし、次の x = new ... した時点で、
x が別の番地を指すようになった。
これ以降 x への変更は main 側にも影響を与えない。
従って main 側で最後に pa を表示させると {1, 2, 3} となる。

#   3.  VB (C#) の場合

VB.NET の場合は、引数を（本来の意味の）参照渡しにすることができる。
仮引数の宣言で、ByRef を付ける。
なお、値渡しにするには ByVal を付けるが、.NET では
省略するとデフォルトで ByVal になっている。

ちなみに VB6 以前は、省略すると ByRef であったので注意が必要。

さらに Structure (構造体) は値型、Class (クラス) は参照型になる。
そこで以下のようなコードを書いてテストしてみる。
