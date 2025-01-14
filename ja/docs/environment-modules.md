# Environment Modules

ABCIでは、[ソフトウェア](system-overview.md#software)で挙げた、さまざまな開発環境、MPI、ライブラリ、ユーティリティ等を提供しています。利用者はこれらのソフトウェアを「モジュール」として、組み合わせて利用できます。

[Environment Modules](http://modules.sourceforge.net/)は、これらのモジュールを利用するのに必要な環境設定を柔軟かつ動的に行う機能を提供します。

## 利用方法 {#usage}

利用者は、`module`コマンドを用いて環境の設定を行えます。

```
$ module [options] <sub-command> [sub-command options]
```

以下にサブコマンドの一覧を示します。

| サブコマンド | 説明 |
|:--|:--|
| list | ロード済みのモジュールの一覧表示 |
| avail | 利用可能なモジュールの一覧表示 |
| show *module* | *module*の設定内容の表示 |
| load *module* | *module*のロード |
| unload *module* | *module*のアンロード |
| switch *moduleA* *moduleB* | モジュールの切り替え（*moduleA*を*moduleB*に置き換える） |
| purge | ロード済みのすべてのモジュールをアンロード（初期化） |
| help *module* | *module*の使用方法の表示 |

## 実行例 {#use-cases}

### モジュールのロード {#load-modules}

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1 cudnn/7.6/7.6.5
```

### ロード済みのモジュールの一覧表示 {#list-loaded-modules}

```
[username@g0001 ~]$ module list
Currently Loaded Modulefiles:
  1) cuda/10.0/10.0.130.1   2) cudnn/7.6/7.6.5
```

### モジュールの設定内容の表示 {#display-the-configuration-of-modules}

```
[username@g0001 ~]$ module show cuda/10.0/10.0.130.1
-------------------------------------------------------------------
/apps/modules/modulefiles/centos7/gpgpu/cuda/10.0/10.0.130.1:

module-whatis	 cuda 10.0.130.1
conflict	 cuda
prepend-path	 CUDA_HOME /apps/cuda/10.0.130.1
prepend-path	 CUDA_PATH /apps/cuda/10.0.130.1
prepend-path	 PATH /apps/cuda/10.0.130.1/bin
prepend-path	 LD_LIBRARY_PATH /apps/cuda/10.0.130.1/extras/CUPTI/lib64
prepend-path	 LD_LIBRARY_PATH /apps/cuda/10.0.130.1/lib64
prepend-path	 CPATH /apps/cuda/10.0.130.1/extras/CUPTI/include
prepend-path	 CPATH /apps/cuda/10.0.130.1/include
prepend-path	 LIBRARY_PATH /apps/cuda/10.0.130.1/lib64
prepend-path	 MANPATH /apps/cuda/10.0.130.1/doc/man
-------------------------------------------------------------------
```

### ロード済みのすべてのモジュールをアンロード（初期化） {#unload-all-loaded-modules-initialize}

```
[username@g0001 ~]$ module purge
[username@g0001 ~]$ module list
No Modulefiles Currently Loaded.
```

### 依存関係のあるモジュールのロード {#load-dependent-modules}

```
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
WARNING: cudnn/7.6/7.6.5 cannot be loaded due to missing prereq.
HINT: at least one of the following modules must be loaded first: cuda/9.0 cuda/9.2 cuda/10.0 cuda/10.1 cuda/10.2
```

依存関係があるため、`cuda/9.0`、`cuda/9.2`、`cuda/10.0`、`cuda/10.1`、`cuda/10.2`のいずれかのモジュールを先にロードしないと`cudnn/7.6/7.6.5`をロードできません。

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
```

### 排他関係にあるモジュールのロード {#load-exclusive-modules}

同一ライブラリの異なるバージョンのモジュールなど、排他関係にあるモジュールは同時に利用することはできません。

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1
[username@g0001 ~]$ module load cuda/10.2/10.2.89
cuda/10.2/10.2.89(7):ERROR:150: Module 'cuda/10.2/10.2.89' conflicts with the currently loaded module(s) 'cuda/10.0/10.0.130.1'
```

### モジュールの切り替え {#switch-modules}

```
[username@g0001 ~]$ module load cuda/10.2/10.2.89
[username@g0001 ~]$ module switch cuda/10.2/10.2.89 cuda/11.0/11.0.3
```

依存関係があると切り替えがうまくできない場合があります。

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
[username@g0001 ~]$ module switch cuda/10.0/10.0.130.1 cuda/10.2/10.2.89
[username@g0001 ~]$ env | grep LD_LIBRARY_PATH
LD_LIBRARY_PATH=/apps/cudnn/7.6.5/cuda10.0/lib64:/apps/cuda/10.2.89/lib64:/apps/cuda/10.2.89/extras/CUPTI/lib64
```

CUDA10.2と、CUDA10.0用のcuDNNがロードされた状態になっています。

以下のように対象のモジュールに依存しているモジュールを事前にアンロード、切り替え後の再ロードする必要があります。

```
[username@g0001 ~]$ module load cuda/10.0/10.0.130.1
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
[username@g0001 ~]$ module unload cudnn/7.6/7.6.5
[username@g0001 ~]$ module switch cuda/10.0/10.0.130.1 cuda/10.2/10.2.89
[username@g0001 ~]$ module load cudnn/7.6/7.6.5
[username@g0001 ~]$ env | grep LD_LIBRARY_PATH
LD_LIBRARY_PATH=/apps/cudnn/7.6.5/cuda10.2/lib64:/apps/cuda/10.2.89/lib64:/apps/cuda/10.2.89/extras/CUPTI/lib64
```

## ジョブスクリプトでの利用方法 {#usage-in-a-job-script}

バッチ利用時のジョブスクリプトで`module`コマンドを利用する場合には、以下のように初期設定を加える必要があります。

sh, bashの場合:

```
source /etc/profile.d/modules.sh
module load cuda/10.0/10.0.130.1
```

csh, tcshの場合:

```
source /etc/profile.d/modules.csh
module load cuda/10.0/10.0.130.1
```
