---
title: "OpenSBI で RISC-V 向けカーネルをブートする"
date: 2023-10-09T17:57:25+09:00
draft: true
---

RISSC-V で OS を作ります．
<https://operating-system-in-1000-lines.vercel.app/ja/> を参考に．
でも納得いっていない部分もあるのでアレンジを加えながらやる（ソースの批判ではなく，簡単のために説明を削っているところがあると思うので気になったら自分で追記・実装するの意）．

# boot

OpenSBI ファームウェアの上で RISC-V 32bit 向けのカーネルをブートする．
