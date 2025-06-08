---
layout: post
title: "Programming Tips: 参照渡しと参照型の値渡し"
date: 2025-01-06
---

# はじめに

一般的なプログラミング言語において関数に引数を渡す時、
値渡しと参照渡しがある。
ここでは詳しい定義は述べないが、簡単に言うと

- 値渡し (call by value)  とは、実引数のコピーを渡す。
- 参照渡し (call by reference)  とは、変数そのものを渡す。

たとえば、以下の疑似コードを考える。

```
func(x):
begin
   x = 2
end

main:
begin
   a = 1
   func(a)
end
```

もし、これが値渡しであれば func の中で行われた引数への変更は
呼び出し元には影響を与えないので a は 1 のままである。

一方、これが参照渡しであれば func の引数 x と main の a は
同じ変数である。よって func の呼び出し後で a は 2 に変わる。

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
これは参照の値渡しである。
あるいは、共有呼び (call by sharing)  とも呼ばれる。

#   2.  C++ の場合

ｍ
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
そこで下記のようなコードを書いてテストしてみる。
なお、テストコードは以下のリポジトリに保管してある。


https://github.com/takahiro-itou/CallByReferenceTest/blob/master/vb/TestApp1/Program.vb

```
    Public Structure PersonV
        Public Name As String
        Public Age As Integer
    End Structure

    Public Class PersonR
        Public Name As String
        Public Age As Integer
    End Class

    Public Sub PassStructByValue(ByVal p As PersonV)
        ' 値型を値渡し
        p.Name = "Alice"
        p.Age = 20

        p = New PersonV()
        p.Name = "Bob"
        p.Age = 25
    End Sub

    Public Sub PassStructByReference(ByRef p As PersonV)
        ' 値型を参照渡し
        p.Name = "Alice"
        p.Age = 20

        p = New PersonV()
        p.Name = "Bob"
        p.Age = 25
    End Sub

    Public Sub PassClassByValue(ByVal p As PersonR)
        ' 参照型を値渡し
        p.Name = "Alice"
        p.Age = 20

        p = New PersonR()
        p.Name = "Bob"
        p.Age = 25
    End Sub

    Public Sub PassClassByReference(ByRef p As PersonR)
        ' 参照型を参照渡し
        p.Name = "Alice"
        p.Age = 20

        p = New PersonR()
        p.Name = "Bob"
        p.Age = 25
    End Sub

    Sub Main(args As String())
        Dim p1 As PersonV
        Dim p2 As PersonR

        Console.WriteLine("CallByValue for Structure")
        p1.Name = "山田太郎"
        p1.Age = 30
        Console.WriteLine("Name = " & p1.Name & ", Age = " & p1.Age)
        PassStructByValue(p1)
        Console.WriteLine("Name = " & p1.Name & ", Age = " & p1.Age)

        Console.WriteLine("CallByReference for Structure")
        p1.Name = "山田太郎"
        p1.Age = 30
        Console.WriteLine("Name = " & p1.Name & ", Age = " & p1.Age)
        PassStructByReference(p1)
        Console.WriteLine("Name = " & p1.Name & ", Age = " & p1.Age)

        Console.WriteLine("CallByValue for Class")
        p2 = New PersonR()
        p2.Name = "山田太郎"
        p2.Age = 30
        Console.WriteLine("Name = " & p2.Name & ", Age = " & p2.Age)
        PassClassByValue(p2)
        Console.WriteLine("Name = " & p2.Name & ", Age = " & p2.Age)
        p2 = Nothing

        Console.WriteLine("CallByReference for Class")
        p2 = New PersonR()
        p2.Name = "山田太郎"
        p2.Age = 30
        Console.WriteLine("Name = " & p2.Name & ", Age = " & p2.Age)
        PassClassByReference(p2)
        Console.WriteLine("Name = " & p2.Name & ", Age = " & p2.Age)
        p2 = Nothing

    End Sub
```

これを実行すると、

(Fig. 1) ![実行結果](https://github.com/takahiro-itou/takahiro-itou.github.io/blob/master/images/CallByReference-001.jpg?raw=true)

となる。

- まず、１個目の値型を値渡ししているケースでは、
  関数には値がコピーされて渡されるので Main とは全くの別物。
  よっていくら仮引数をいじっても Main には影響を与えていない。
- ２個目は値型を参照渡ししている。
  参照渡しであるから Main の変数と、仮引数 p は同じ変数であるので、
  関数内の最後の状態と同じになる。つまり Bob になる。
- ３個目は、参照型を値渡ししているケース。
  これは、先ほどまで python や C++ の例と同じである。
  p が指している先のオブジェクトを弄るだけならその影響を受けるので、
  まず Alice になる。
  その後 Bob インスタンスを作って p に代入した時点で、
  Main とは別物になる。よって Main の p2 は、Alice のまま Bob にはならない。
- ４個目は、参照型を参照渡ししている。
  この場合は p = New ... で新しいインスタンスを代入しても
  そもそも p が参照渡しなので、Main の変数と仮引数 p は同じ変数。
  よって Main の p2 も、
  その新しいインスタンスを指すように参照が更新される。
  よって、このケースは２番目と同じで Bob になる。


#   参考

- https://ja.wikipedia.org/wiki/%E8%A9%95%E4%BE%A1%E6%88%A6%E7%95%A5
