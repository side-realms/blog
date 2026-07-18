---
title: "自作 OS でカーネルパニック"
date: 2023-10-13T17:57:25+09:00
draft: false
---

次はカーネルパニックを実装する．
昨日は assert を実装したのでほとんど同じように実装できる．

多分 ##__VA__ARGS__ が assert とは違うところになる．
assert は真偽だけを判定すればよいが， PANIC は真偽だけではなく内部の引数もそのまま食う必要があるので，可変長引数として必要．

```C
#define assert(expr) \
    do { \
        if (!(expr)) { \
            printf("Assertion failed: %s at %s:%d\n", #expr, __FILE__, __LINE__); \
            for (;;); \
        } \
    } while (0)

#define PANIC(fmt, ...)                                                        \
    do {                                                                       \
        printf("PANIC: %s:%d: " fmt "\n", __FILE__, __LINE__, ##__VA_ARGS__);  \
        while (1) {}                                                           \
    } while (0)
```

でも，assert も結局パニックを呼べばいいので，以下のようにする

```C
#define assert(expr) \
    do { \
        if (!(expr)) PANIC("Assertion failed: %s", #expr); \
    } while (0)
```