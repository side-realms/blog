---
title: "OpenSBI で RISC-V 向けカーネルをブートする"
date: 2023-10-10T17:57:25+09:00
draft: false
---

RISSC-V で OS を作ります．
<https://operating-system-in-1000-lines.vercel.app/ja/> を参考に．
でも納得いっていない部分もあるのでアレンジを加えながらやる（ソースの批判ではなく，簡単のために説明を削っているところがあると思うので気になったら自分で追記・実装するの意）．

# boot

OpenSBI ファームウェアの上で RISC-V 32bit 向けのカーネルをブートする．
とりあえず言われたとおりに実装すれば OK

## OpenSBI

- SBI は RISC-V の S モードで動作する OS と，M モードで動作する FW をインターフェースする仕様
  - M-mode：FW など最上位特権
  - S-mode：Kernel など OS の特権
  - U-mode：アプリなどユーザー空間
- Linux は M-mode のレジスタなどに直接アクセスすることができないので，Linux が割り込みなどをしたいときは SBI 経由で FW に依頼する必要がある
  - x86 における Ring みたいなもの
  - OS が全権を握るわけではないところが面白い
- OpenSBI は，SBI の仕様に準拠したリファレンス実装
