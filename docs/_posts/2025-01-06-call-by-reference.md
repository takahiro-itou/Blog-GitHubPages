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
