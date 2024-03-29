# BADotNet

**_Bifurcation Analysis with openMP_**

入れ子構造を持つ粒子群最適化 (Nested-Layer Particle Swarm Optimization, NLPSO) を用いて離散時間力学系の分岐パラメータ曲線を導出するコンソールアプリです。

.NET 6 を用いて開発・実行していますが、実行環境はフレームワークに依存しません。gcc を含む C 言語の実行環境があれば、多くのプラットフォームで他のソフトウェアに依存せず利用できます。

- [注意](#注意)
- [動作確認済みの環境](#動作確認済みの環境)
- [導入](#導入)
- [クイックスタート](#クイックスタート)
- [使い方](#使い方)
  - [実行](#実行)
    - [２回目以降](#２回目以降)
  - [探索に用いるパラメータの設定](#探索に用いるパラメータの設定)
  - [探索の実行](#探索の実行)
- [仕様詳細](#仕様詳細)
  - [必要な系の定義（用意する c ソースファイル）について](#必要な系の定義用意するcソースファイルについて)
  - [ヘッダファイルについて](#ヘッダファイルについて)
  - [パラメータの詳細](#パラメータの詳細)
  - [探索アルゴリズム（NLPSO）について](#探索アルゴリズムnlpsoについて)
  - [openMP を用いた並列化（適切なスレッド数）について](#openmpを用いた並列化適切なスレッド数について)
- [バージョン履歴](#バージョン履歴)
- [既知の不具合](#既知の不具合)
- [権利表記](#権利表記)
- [文献](#文献)

## 注意

- 相対パスを用いたディレクトリおよびファイルの生成を行います。同名のファイルが存在した場合確認なしに上書きするため、必ず空のディレクトリに展開し、実行ファイルが存在するディレクトリで実行してください。
- `gcc` コマンドで gcc を呼び出せるシェルでのみ動作します。
  - clang では動作しません。`gcc` コマンドが clang を呼ぶようになっている場合は動作しませんのでご注意ください（特に Mac ユーザー）。
- intel CPU (Core シリーズ) でのみ動作確認しています。

## 動作確認済みの環境

- Windows 11 Pro バージョン 21H2 ビルド 22000.556

  - gcc version 8.1.0 (x86_64-posix-sjlj-rev0, Built by MinGW-W64 project)

- macOS Monterey 12.3.1 (MacBook Pro / intel Core i7)

  - gcc version 11.2.0 (Homebrew GCC 11.2.0_3)

- Ubuntu 18.04.6 LTS (Desktop)

  - gcc version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)

- Ubuntu 20.04.3 LTS (WSL)
  - gcc version 9.3.0 (Ubuntu 9.3.0-17ubuntu1~20.04)

※ intel CPU のみ動作を確認しています

## 導入

お使いの OS に合わせて zip ファイルをダウンロードしてください。インストールは不要です。Linux 向けのものは CentOS、Debian、Fedora、Ubuntu、およびこれらの派生ディストリビューションで動作することを期待しています。

## クイックスタート

```
$ ./BADotNet Sample.c

>> set period 2 3 5

>> run
```

Circle map[1] の 2/3/5 周期点における周期倍分岐パラメータと分岐図を得られます。成果物は out ディレクトリ内に生成されています。

## 使い方

シェル CLI から実行してください。コマンドライン引数として、.c ファイルまたは .xml ファイルのパスが必要です。

### 実行

解析対象のシステム定義を c ソースファイルとして与える必要があります。ソースファイルの詳細な記法については後述します。同梱されている Sample.c には Circle map の定義が記述されています。

**Sample.c はアプリケーションの実行フォルダに毎回上書き生成されます。編集した場合は必ず別名で保存してください。**

Linux

```
$ ./BADotNet Sample.c
```

Mac

```
$ ./BADotNet Sample.c
```

Windows (Power Shell)

```
> ./BADotNet.exe Sample.c
```

正常に実行できた場合、\>> と表示される入力待ち状態になります。

Period の設定を行い、`run` コマンドで探索を実行します。

```
>> set period 2 3 5
// 2周期点、3周期点、5周期点で分岐パラメータを探索

>> run
```

正常に探索が終了した場合、実行ディレクトリ直下に out ディレクトリが生成され、探索結果を記録した csv ファイルと、それらから生成された 2 パラメータ分岐図が html 形式で生成されています。

#### ２回目以降

正常に探索プロセスを実行できた場合、アプリケーションと同じディレクトリに xml ファイルが生成されているはずです。xml ファイルをコマンドライン引数で与えて実行すると、前回そのシステムの解析に用いたパラメータをそのまま使用することができます（この場合も前回使用した c ソースファイルは必要です）。

```
$ ./BADotNet Sample.xml

// 作業ディレクトリに Sample.c が必要
```

### 探索に用いるパラメータの設定

`get` コマンドで設定可能なパラメータの一覧を取得できます。

```
>> get
// =>
//  Variable   |  key        |  value
// ------------+-------------+------------
//  Target     |  target     | Sample
//  NumTrial   |  trial      | 100
//  Period     |  period     |
//  Mu         |  mu         | -1
//  Population | (mbif, mpp) | (30, 30)
//  Tmax       | (tbif, tpp) | (600, 300)
//  Criterion  | (cbif, cpp) | (0.001, 1E-10)
//  OpenMP     |  openmp     | True (max_threads: 16)
```

`>> get [key]` で指定したパラメータの値のみを取得できます。

```
>> get mbif
// => 30
```

`>> set [key] [value]` でパラメータの値を設定することができます。

```
>> set tpp 100
>> get Tmax
// => (600, 100)
```

### 探索の実行

`run` コマンドで探索プロセスが開始されます。

## 仕様詳細

### 必要な系の定義（用意する c ソースファイル）について

必要なもの

- 分岐パラメータの探索範囲 `LMAX[l1_max, l2_max]`, `LMIN[l1_min, l2_min]`
- 周期点の探索範囲 `XMAX[x1_max, x2_max, ... , xn_max]`, `XMIN[x1_min, x2_min, ... , xn_min]`
- 進化関数（差分方程式） `next_x()`

関数 `void next_x(double *x, double *l)` は、状態変数ベクトル `x` をパラメータベクトル `l` を用いて更新するように記述してください。`x` の各要素は破壊的に書き換えられなければなりません。また、`l` の各要素が変更されてはいけません。

任意のユーザ関数を追加できますが、上で示した識別子（変数名及び関数名）は変更できません。Sample.c を別名で保存して直接編集することをおすすめします。

なお、探索するパラメータは 2 つを想定していますが 3 つ以上のパラメータを探索することも可能です。ただし、出力される分岐図は常に 1 つめのパラメータを横軸に、2 つめのパラメータを縦軸にとり、3 つ目以降は無視されます。

### ヘッダファイルについて

実行環境に用意されたヘッダファイルは通常通り使用可能です。自作のヘッダファイルを使用する場合、アプリケーションの実行ディレクトリ**ではなく**、直下の csources ディレクトリに保存してください。

### パラメータの詳細

| パラメータ | key            | 既定値          | `set` コマンドで有効な形式 | 説明                                                                                                                                |
| :--------- | :------------- | :-------------- | :------------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| Target     | `target`       | (Sample)        | 数値始まりでない文字列     | 対象となる系の名前です。探索には影響しませんが、使用する c ソースのファイル名と一致している必要があります。原則自動で設定されます。 |
| NumTrial   | `trial`        | `100`           | 正の整数                   | 導出する分岐パラメータの数です。複数の Period を指定した場合、それぞれについてこの数ずつ導出します。                                |
| Period     | `period`       | なし            | 正の整数列（空白区切り）   | このパラメータで指定した周期の周期点における分岐パラメータを探索します。                                                            |
| Mu         | `mu`           | `-1`            | `1` または `-1`            | 特性乗数です。-1 なら周期倍分岐点を、1 ならサドルノード分岐点を探索します。                                                         |
| Population | `mbif` / `mpp` | `(30, 20)`      | 正の整数                   | PSO の粒子数です。それぞれ分岐パラメータを探索する PSO と周期点を探索する PSO に対応します。                                        |
| Tmax       | `tbif` / `tpp` | `(1000, 100)`   | 正の整数                   | PSO の最大更新回数です。それぞれ分岐パラメータを探索する PSO と周期点を探索する PSO に対応します。                                  |
| Criterion  | `cbif` / `cpp` | `(1e-3, 1e-10)` | 正の実数                   | PSO の探索終了判定に用いる閾値です。それぞれ分岐パラメータを探索する PSO と周期点を探索する PSO に対応します。                      |
| OpenMP     | `openmp`       | `16` (`true`)   | 整数                       | openMP で用いる最大のスレッド数です。1 以下の値が設定された場合は openMP を使用しません (`false`) 。                                |

### 探索アルゴリズム（NLPSO）について

大部分は [1] に基づき実装されています。ただし、PSOpp の評価関数には [1] の (21) 式で示されている Fpp(zp) の 2 乗を用いています。これにともなって、パラメータ `cpp` の既定値が `1e-10` となっています。

### openMP を用いた並列化（適切なスレッド数）について

- 典型的には、使用している CPU の物理コア数に等しい値を設定することでパフォーマンスが最高になります。

## バージョン履歴

2022/04/11 v0.0.3

## 既知の不具合

- Windows 環境において、分岐図生成後のブラウザ起動に失敗する
- Ubuntu 環境において、ハンドルされていない例外で終了した場合に実行した端末がハングする
  - 他の.NET アプリでも類似した事象が起きることがある模様。.NET の不具合である可能性あり

## 権利表記

このアプリケーションは MIT License のもとで提供されます。
同梱の license.txt を参照してください。

このアプリケーションは以下のソフトウェアの複製を含みます。

- Plotly.NET
  - Copyright 2020 Timo Mühlhaus
  - license: MIT License https://opensource.org/licenses/MIT

## 文献

[1] H. Matsushita, H. Kurokawa, and T. Kousaka, "Bifurcation analysis by particle swarm optimization," NOLTA, vol. 11, Oct. 2020

---

(c) 2022 Tomo Hasegawa
