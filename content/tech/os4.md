---
title: "自作 OS で例外処理"
date: 2023-10-14T17:57:25+09:00
draft: false
---

## 例外

- 例外は，CPU がこの命令を実行できない，と判断したときに自動的に発生する
  - ゼロ除算
  - 不正なメモリアドレスへのアクセス
  - 無効な命令の発行
- 例外が発生すると，CPU は今やっていた処理を中断して，別の場所にジャンプする

### CSR

- ここまでは a0 みたいな汎用レジスタを使っていたが，今回初めて CSR を使う
  - stvec: 例外ハンドラのアクセスで，例外発生時ここのジャンプする
  - scause: 例外の種類
  - stval: 例外の追加情報（不正アクセスのアドレス）
  - sepc: 例外が発生した命令のアドレス（復帰先）
  - sstatus: CPU の状態フラグ
  - sscratch: OS が一時退避用に自由に使える CSR
- これは csrr と csrw でしかアクセスできない

### 例外が起きた瞬間に CPU がやること

- 例外発生時，CPU が C のコードを使わずに HW 自身で対応する
  1. 現在の PC を sepc に保存（後から戻ってくる場所）
  2. 例外の種類を scause に書く
  3. 付加情報を stval に書く
  4. sstatus に元の動作モードなどを保存
  5. PC を stvec の値に変更（例外ハンドラへのジャンプ）
  6. sscratch レジスタを使って，汎用レジスタを保存し，例外の種類に応じた処理を行う
  7. 処理を済ませると sret を呼び出して，例外発生場所から実行再会
- ここからも分かる通り，汎用レジスタを正しく保存・復元することが重要．

```C
void kernel_entry(void) {
    __asm__ __volatile__(
        "csrw sscratch, sp\n"
        "addi sp, sp, -4 * 31\n"
        "sw ra,  4 * 0(sp)\n"
        "sw gp,  4 * 1(sp)\n"
        "sw tp,  4 * 2(sp)\n"
        "sw t0,  4 * 3(sp)\n"
        "sw t1,  4 * 4(sp)\n"
        "sw t2,  4 * 5(sp)\n"
        "sw t3,  4 * 6(sp)\n"
        "sw t4,  4 * 7(sp)\n"
        "sw t5,  4 * 8(sp)\n"
        "sw t6,  4 * 9(sp)\n"
        "sw a0,  4 * 10(sp)\n"
        "sw a1,  4 * 11(sp)\n"
        "sw a2,  4 * 12(sp)\n"
        "sw a3,  4 * 13(sp)\n"
        "sw a4,  4 * 14(sp)\n"
        "sw a5,  4 * 15(sp)\n"
        "sw a6,  4 * 16(sp)\n"
        "sw a7,  4 * 17(sp)\n"
        "sw s0,  4 * 18(sp)\n"
        "sw s1,  4 * 19(sp)\n"
        "sw s2,  4 * 20(sp)\n"
        "sw s3,  4 * 21(sp)\n"
        "sw s4,  4 * 22(sp)\n"
        "sw s5,  4 * 23(sp)\n"
        "sw s6,  4 * 24(sp)\n"
        "sw s7,  4 * 25(sp)\n"
        "sw s8,  4 * 26(sp)\n"
        "sw s9,  4 * 27(sp)\n"
        "sw s10, 4 * 28(sp)\n"
        "sw s11, 4 * 29(sp)\n"

        "csrr a0, sscratch\n"
        "sw a0, 4 * 30(sp)\n"

        "mv a0, sp\n"
        "call handle_trap\n"

        "lw ra,  4 * 0(sp)\n"
        "lw gp,  4 * 1(sp)\n"
        "lw tp,  4 * 2(sp)\n"
        "lw t0,  4 * 3(sp)\n"
        "lw t1,  4 * 4(sp)\n"
        "lw t2,  4 * 5(sp)\n"
        "lw t3,  4 * 6(sp)\n"
        "lw t4,  4 * 7(sp)\n"
        "lw t5,  4 * 8(sp)\n"
        "lw t6,  4 * 9(sp)\n"
        "lw a0,  4 * 10(sp)\n"
        "lw a1,  4 * 11(sp)\n"
        "lw a2,  4 * 12(sp)\n"
        "lw a3,  4 * 13(sp)\n"
        "lw a4,  4 * 14(sp)\n"
        "lw a5,  4 * 15(sp)\n"
        "lw a6,  4 * 16(sp)\n"
        "lw a7,  4 * 17(sp)\n"
        "lw s0,  4 * 18(sp)\n"
        "lw s1,  4 * 19(sp)\n"
        "lw s2,  4 * 20(sp)\n"
        "lw s3,  4 * 21(sp)\n"
        "lw s4,  4 * 22(sp)\n"
        "lw s5,  4 * 23(sp)\n"
        "lw s6,  4 * 24(sp)\n"
        "lw s7,  4 * 25(sp)\n"
        "lw s8,  4 * 26(sp)\n"
        "lw s9,  4 * 27(sp)\n"
        "lw s10, 4 * 28(sp)\n"
        "lw s11, 4 * 29(sp)\n"
        "lw sp,  4 * 30(sp)\n"
        "sret\n"
    );
}
```