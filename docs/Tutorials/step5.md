# 【第五回】機器の自作

- 本ステップでできるようになること  
    自分で用意した機器をpower_simulatorに実装できるようになる

## 実装しなければならないこと

- componentクラスの継承
    - componentクラスの抽象メソッド4つを実装

## 解説：抽象メソッドについて
新しいコンポーネントのクラスを定義する際は、以下のような関数をmethodの中に定義しておく必要がある。  
実際にcomponentクラスのインスタンスとしてpower_simulator内に既に定義されている`generator.m` や `load_~.m` などのファイルを見ると、これらの関数が定義されていることが確認できるはずである。

- __x = initialize(V, I, net)__  
    潮流計算で決まったV,Iを受け取って初期化処理を行い平衡状態を導出するためのメソッド
    - 入力引数
        - `V`：潮流計算の結果得られる電圧の平衡点(複素数)
        - `I`：潮流計算の結果得られる電流の平衡点(複素数)
        - `net`：power_networkのインスタンス
    - 出力因数
        - `x`：指定された平衡点に対応する平衡状態
    
        netにネットワークが渡されるので、その情報を使っても良い。generator_AGCではnet.omega0(基準周波数)だけ使っている。
  

- __nx = get_nx()__  
    クラス内に定義されている状態`x`の次数を求めるためのメソッド
   - 出力引数
        - `nx`：状態変数の次数
- __nu = get_nu()__  
    クラス内に定義されている状態`u`の次数を求めるためのメソッド
    - 出力引数
        - `nu`：入力変数の次数
- __[dx,I] = get_dx_I(t, x, v, u)__  
    入力を印加した時の状態の微分と電流を取得するためのメソッド
    - 入力引数
        - `t`：時刻
        - `x`：自分の状態
        - `v`：バスの電圧
        - `u`：入力信号
    - 出力引数
        - `dx`：入力が与えられた時の$\dot{x}$
        - `I`：電流（実部と虚部のベクトル）

## 実装した方が望ましいメソッド(オーバーライド推奨)
以下の関数は`component.m`の内部で定義されている関数であるため、componentクラスのインスタンスとして新しい自作の機器を作成したとき自動的に継承されるが、各機器によって関数の内容を再定義する方が望ましい。

- __[A, B, C, D, BV, DV, R, S] = get_linear_matrix()__  
    以下の式のような線形化したシステムの行列A，B，C，D，BV，DVを返すメソッド。

    $$
    \begin{matrix}
    \begin{align}
  　\dot{x}　&=A(x-x^*)+Bu+B_V(V-V^*)＋Rd\\
    I-I^*&=C(x-x^*)+Du+D_V(V-V^*)\\
    ｚ&＝S(x^*-x)
    \end{align}
    \end{matrix}
    $$
  ただしR,Sは制御系設計のために使われる外乱ポートと評価出力ポートであり機器の性質ではないため消しても構わない。  
  
- __[dx, I] =get_dx_I_linear(obj, t, x, v, u)__  
    上の「解説(抽象メソッド)」の章で紹介したget_dx_Iの線形化バージョンである。  
    現状、get_linear_matrixを実装しても自動的に反映されない。
    - `get_dx_I_linear`と`get_linear_matrix`を実装しない場合、initializeの中で  
        `obj.numerical_diff(x, [Vr; Vi]);`
    を呼ぶと、平衡点周りでの数値積分によって２つのメソッドがつかえるようになる。  
  
