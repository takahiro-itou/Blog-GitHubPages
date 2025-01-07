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
