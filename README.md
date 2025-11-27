# What is this

This provides a build configuration for the C implementation of [catch22](https://github.com/DynamicsAndNeuralSystems/catch22) in Zig.

# Build instructions

Run the following command:

```bash
zig build
```

---

# ビルドプロセスの解説

## 概要

このリポジトリは、時系列データ解析ライブラリ [catch22](https://github.com/DynamicsAndNeuralSystems/catch22) の C 実装を **Zig ビルドシステム** を使用してビルドするためのプロジェクトです。

## プロジェクト構成

```
catch22-test/
├── build.zig          # Zig ビルド設定ファイル
├── README.md          # このファイル
├── .gitignore         # Git 除外設定
└── src/               # ソースコード
    ├── main.c         # メインエントリーポイント
    ├── main.h
    ├── *.c / *.h      # catch22 の各機能モジュール
    └── runAllTS.sh    # テスト用シェルスクリプト
```

## ビルドシステム詳細

### build.zig の構造

`build.zig` ファイルは Zig のビルドシステム設定を定義しています：

```zig
const std = @import("std");

pub fn build(b: *std.Build) void {
    // 1. ターゲットプラットフォームと最適化オプションの設定
    const target = b.standardTargetOptions(.{});
    const mode = b.standardOptimizeOption(.{});

    // 2. モジュールの作成（C ライブラリとしてリンク）
    const module = b.addModule("catch22", .{
        .target = target,
        .optimize = mode,
        .link_libc = true,  // libc をリンク
    });

    // 3. C ソースファイルの追加（23ファイル）
    module.addCSourceFiles(.{
        .files = &.{
            "src/butterworth.c",
            "src/CO_AutoCorr.c",
            // ... 他21個のソースファイル
        },
    });

    // 4. 実行可能ファイルの作成
    const exe = b.addExecutable(.{
        .name = "catch22",
        .root_module = module,
    });

    // 5. インストールターゲットの設定
    b.installArtifact(exe);
}
```

### ビルドステップの詳細

#### ステップ 1: ターゲット設定
```zig
const target = b.standardTargetOptions(.{});
const mode = b.standardOptimizeOption(.{});
```
- `target`: ビルド対象のプラットフォーム（OS、アーキテクチャ）を指定
- `mode`: 最適化レベル（Debug, ReleaseSafe, ReleaseFast, ReleaseSmall）を指定

コマンドラインから指定可能：
```bash
zig build -Dtarget=x86_64-linux-gnu -Doptimize=ReleaseFast
```

#### ステップ 2: モジュール作成
```zig
const module = b.addModule("catch22", .{
    .target = target,
    .optimize = mode,
    .link_libc = true,
});
```
- `catch22` という名前のモジュールを作成
- `link_libc = true`: 標準 C ライブラリをリンク（`math.h`, `stdio.h` 等の使用に必要）

#### ステップ 3: ソースファイル追加
23個の C ソースファイルがビルド対象として追加されます：

| カテゴリ | ファイル | 説明 |
|---------|----------|------|
| **メイン** | `main.c` | プログラムのエントリーポイント |
| **統計機能** | `stats.c`, `DN_*.c` | 分布・統計計算 |
| **自己相関** | `CO_AutoCorr.c`, `CO_*.c` | 自己相関分析 |
| **FFT** | `fft.c`, `SP_Summaries.c` | フーリエ変換・スペクトル分析 |
| **フィルタ** | `butterworth.c` | バターワースフィルタ |
| **その他** | `histcounts.c`, `splinefit.c` 等 | ヘルパー関数 |

#### ステップ 4: 実行可能ファイル作成
```zig
const exe = b.addExecutable(.{
    .name = "catch22",
    .root_module = module,
});
```
- `catch22` という名前の実行可能ファイルを生成

#### ステップ 5: インストール設定
```zig
b.installArtifact(exe);
```
- ビルド成果物を `zig-out/bin/` にインストール

## ビルドコマンド

### 基本ビルド
```bash
zig build
```

### オプション指定ビルド
```bash
# リリースビルド（最適化あり）
zig build -Doptimize=ReleaseFast

# デバッグビルド
zig build -Doptimize=Debug

# クロスコンパイル（例: Windows 向け）
zig build -Dtarget=x86_64-windows-gnu
```

### 出力ファイル
ビルド成功時、実行可能ファイルは以下の場所に生成されます：
```
zig-out/bin/catch22
```

## 使用方法

```bash
# ビルド
zig build

# 実行
./zig-out/bin/catch22 <入力ファイル> [出力ファイル]
```

入力ファイルは1行に1つの数値を含む時系列データです。

## 必要な環境

- **Zig**: バージョン 0.11.0 以降推奨
- **OS**: Linux, macOS, Windows (クロスコンパイル可能)

## Zig を使用する利点

1. **クロスコンパイル**: 単一のビルド環境から複数プラットフォーム向けにビルド可能
2. **簡潔な設定**: `build.zig` 一つでビルド設定が完結
3. **C との互換性**: C コードをそのまま使用可能
4. **再現性**: ビルド環境に依存しない一貫したビルド結果
