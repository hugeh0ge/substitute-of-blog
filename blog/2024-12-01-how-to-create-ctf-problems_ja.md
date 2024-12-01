# はじめに

この記事は[CTF Advent Calendar 2024](https://adventar.org/calendars/10469)の1日目の記事です。期限を過ぎてしまってごめん（12/1 24時20分投稿）。
久々にCTFのアウトプットでもするかということで、自分流の作問のやり方みたいなものを書いてみようと思います。
インターネット上にwriteupは数あれど、あまり作問のやり方について具体的に言及している記事は少ないんじゃないでしょうか。
ここでは、主に自分が作問した問題を題材として、いくつかの作問のパターンを紹介します（網羅的とはいえません）。
自分が作問するメインの問題ジャンルはpwnであるため、細かいテクニカルな部分は、pwnにしか当てはまらないことがあります。
とはいえ、webでは同じ考え方を適用できることが多いです。cryptoや他のジャンルは微妙かも。

また当然、紹介する問題については設計思想や解法について言及するため、ネタバレを含みます。
以下に題材とする問題を挙げるので、解きたい問題がある場合は先に解いてから読み始めてください：

- IERAE CTF 2024 pwn (warmup) "This is warmup"
- CODE BLUE CTF 2018 Quals pwn (easy) "Watch Cats"
- Burning CTF 2016 pwn (medium) "craSH (part1)"
- IERAE CTF 2024 pwn (easy) "Command Recorder"
- Burning CTF 2016 pwn (medium) "Ninja no Aikotoba"
- CODE BLUE CTF 2017 pwn (medium) "nonamestill"
- WCTF 2017 pwn (hard) "Under Debugging Revenge"
- IERAE CTF 2024 pwn (hard) "New Under The Sun"
- IERAE CTF 2024 pwn (medium) "Intel CET Bypass Challenge"
- CODE BLUE CTF 2018 Finals pwn (medium) "fliqpy"
- Asian Cyber Security Challenge 2024 pwn (easy) "fleeda"

# テーマを決める

作問の各パターンを紹介する前に話しておくべきこととして、どんな問題でも共通して必ずやらないといけないことがあります。それは問題のテーマを決めることです。
すべてのCTFの問題は、なぜその問題を作ったのか、その問題を通じてどういうことを体験してほしいか（必ずしも勉強ではない）ということが説明できる状態であるべきです。
作問者がこういったことを説明できない状態で作られた問題は、往々にして面白くない問題やひどい問題、CTFとして成立していない問題になりやすいです。

これは、難しい問題に限られた話ではありません。
たとえどんなに簡単な問題であっても、達成できているかを省みるべきものです。
たとえば、このあと出てくる問題 "This is warmup" のテーマは、「pwnの初心者を対象として、整数値オーバーフローという脆弱性がどのような原理で発生するかを理解してもらう。このとき、整数値オーバーフローを深く理解してもらうため、適当に大きな数を入力するだけでは解けないように、間違った整数値オーバーフローのチェックを含める。」と説明できます。

テーマの設定は、必ずしも一番初めにやらなければならないわけではないことを注意しておきます。
作問自体の取っ掛かり自体は、もっと要素技術のようなピンポイントな部分であっても問題ありません。
ただし、最終的にできあがった問題がなんなのか、ということを説明できるように作問を着地させるべきということです。

# 問題のコアを決める

テーマとは独立して、問題のコア・本質になる要素を決める必要があります。
これが知られていないものや非自明なものになっているほど、難しさや面白さが比例する傾向があります。
前述したように、テーマを決めてから問題のコアを決めてもよいですし、問題のコアを決めてから逆算する形でテーマを決めても良いと思います。

問題のコアは、大きく分けて3つの分類があります。
1つ目は脆弱性、2つ目は解法、3つ目は対象です。

脆弱性をコアにした問題とは、一見、何も問題がないように見えるコードに脆弱性が含まれるようなものを指します。
解法をコアにした問題とは、脆弱性がわかってもなお、その悪用が簡単ではないようなものを指します。
対象をコアにした問題とは、難しいのは脆弱性でも解法でもよく（極論、それらはまったく難しくなくてもよい）、ただ単にこれまで問題の題材とされていなかったような対象を攻略するような問題を指します。

基本的に私が作問するときは、1つコアとなるようなアイデアや観察をもとに、これら3つのうちどれかを出発点として、他の部分を肉付けしていくようなイメージで進めていきます。
もちろんこの過程で、難易度の調整や完成度の観点から、コアを複数混ぜて複合的な問題とすることもあります。
このような、コアになりそうなアイデアや観察は、ときとして簡単に発見できるようなものではないので、私の場合は、発見したときに作問アイデアノートにメモするようにしています。

## 1. 脆弱性をコアとする場合

脆弱性をコアにする場合には、コードに含まれる脆弱性が多少は非自明であることが求められます。
この非自明度合いというのは、対象とする挑戦者のスキルレベルに合わせて調整してよいです。
つまり、熟練のCTFプレイヤーやセキュリティエンジニアからすれば自明な実装ミスであっても、初心者からすると初見では見逃してしまうようなものであればOKであることもあります。

このような脆弱性をコアとする問題も、いくつかの小分類にカテゴライズすることができるため、それぞれ紹介します。

### パターン1-a: ありがちなミス

現実のプロダクトなどにおいてもよく見られるような実装ミスを題材とするパターンです。
"ありがちな"ミスである以上、この要素だけで難しい問題を作ることは難しく、主に初心者から中級者までを対象とした問題でよく使います。

#### 問題例1-a-1: IERAE CTF 2024 pwn (warmup) "This is warmup"

C言語では、言語の仕様上、整数値オーバーフローが非常に簡単に起きてしまいます。
そしてその対策は難しく、OSSなどにおいても頻繁に発見される脆弱性であるといえます（最近も7zipで見つかっていましたね）。
この問題はIERAE CTFのwarmup問題、すなわちpwnジャンルの中で一番初めに解かれるように設計した問題になります。
そのため、脆弱性は2整数を掛けただけで発生する単なる整数値オーバーフロー、フラグはプログラムをSEGVさせるだけで取れるようにしています。
ただし、初心者に整数値オーバーフローを理解してもらう上で、無限の猿定理がごとく、「適当に数を入力したら解けた」という問題にするのは望ましくないと考えました。
そのため、整数値オーバーフローが引き起こせる数の条件を制限しています。

これ以上の詳細は[公式writeup](https://gmo-cybersecurity.com/blog/ierae-ctf-2024-writeup-pwn/#a)を見るといいでしょう。

#### 問題例1-a-2: CODE BLUE CTF 2018 Quals pwn (easy) "Watch Cats"

同様に整数値オーバーフローを題材とした問題として、"Watch Cats"があります。
Watch Dogsというゲームのミニゲームを模したプログラムになっており、ゲームをクリアするとフラグが得られます（もちろん、脆弱性をつかないとクリアできないように設計されている）。

![Watch Catsの実行画面。Dragon Sectorのq3kのwriteupより](https://blogger.googleusercontent.com/img/b/R29vZ2xl/AVvXsEhcqD1WeBNP2VYozXk-erXKcuGOV6yH7rtrwMzkl0xy5VjCsPqZDaUINA3W0mJkBH29gQI1zNS-IkP_dh6vU8RmccSeJ5ybeGwxFx-K6xZPnIXxn5cWz59vwAR6bTlPttUsaEwx_cxgx_Q/s640/2018-07-29-143020_790x572_scrot.png "Watch Catsの実行画面。Dragon Sectorのq3kのwriteupより")

この問題では、以下の`char`型の変数`direction`に対するインクリメントにおいて整数値オーバーフローが起きます：

```c
      scanf("%d", &idx);
      if (idx < 0 || num_joints <= idx) {
        puts("Idiot");
        exit(0);
      }
      ++joints[idx].direction; // 128回やると負数になる
...
      struct joint &jo = joints[idx];
      const struct joint_type &ty = joint_types[jo.type];

      int k;
      for (k=0; k<4; k++) {
        if (jo.cables[k] == v) break;
      }

/*
  剰余を取られると3以下になってしまうが、インクリメントごとに剰余を取っているわけではないため、
  剰余が発生する前にインクリメントを128回やると負数になる。
  C言語では負数に対して剰余をとっても負数のままなので、
  負数になったあとこれをインデックスとして使うと配列外参照が起きる
*/
      jo.direction %= 4;
```

掛け算で整数値オーバーフローが発生する場合に比べて、インクリメントで発生する場合のほうが非自明度合い（一瞬見逃してしまう可能性）が高いというのは同意してもらえるでしょう。
このように、同じ脆弱性を題材とする場合でも、コードの書き方や肉付けの仕方によって、雰囲気を変えることが可能です。

ちなみにこの問題においても、適当にランダムな入力を繰り返すだけでは、脆弱性を発見できないようになっています。具体的には、毎回ではないものの頻繁に剰余が取られるようになっているため、意図的に剰余が起こらないようにセットアップをした後、インクリメントしないことには、脆弱性は発現しません。

#### 問題例1-a-3: Burning CTF 2016 pwn (medium) "craSH (part1)"

簡易的なシェルが与えられるので、SEGVを引き起こせばフラグがもらえるという問題でした（part 2はそこから任意のコード実行）。
一見どこにも脆弱性がないように見えますが、`cat`コマンドの実装に脆弱性があります。

```c
struct file {
    size_t len;
    char *data;
};

...

// outputにはリダイレクトがある場合にはリダイレクト先のファイルを表すインスタンス、
// ない場合にはstdoutを表す構造体インスタンスが与えられる
void args_cat(char *args[], size_t num, struct file *output) {
    size_t i;
    size_t sz = 0;
    char *written_pos;

    for (i=1; i<num; i++) {
        struct file *f = get_file(args[i]);
        if (f == NULL) {
            printf("%s: File Not Found.\n", args[i]);
            continue;
        }

        sz += f->len;
    }

    if (output->len < sz) {
        output->data = (char*)realloc(output->data, sizeof(char) * (sz+1));
    }
    output->len = sz;

    if (output->data == NULL) {
        fprintf(stderr, "[*] Memory Error.\n");
        exit(-1);
    }

    written_pos = output->data;
    for (i=1; i<num; i++) {
        struct file *f = get_file(args[i]);

        if (f == NULL) continue;
        memcpy(written_pos, f->data, f->len);
        written_pos += f->len;
    }
}
```

このコードだけでは脆弱性がどこにあるか分からないのが普通だと思いますが、以下のSEGVを引き起こすPoCを見ると脆弱性は明らかでしょう：

```
$ echo aaaaaaaaaaaaaaaa > a
$ cat a a > a
```

このように、ある種ロジック的な、コーナーケースを考えるようにすることでも、この種のコアを作ることができます。

### パターン1-b: 仕様に違反した使い方

多くのプログラミング言語やライブラリにおいて、仕様に違反した使い方をした場合、結果は未定義とされ、現実には脆弱性を引き起してしまうことは少なくありません。
このパターンではあえて何かを仕様に反する使い方をすることで、意図的に脆弱な状況を作り出します。
典型的には、C言語のライブラリ関数などを対象とすることが多いです。
このため、Linuxのライブラリ関数のmanの説明や、フレームワークやライブラリのマニュアルを読み込む癖をつけておくと、このパターンの作問が容易になります。

#### 問題例1-b-1: IERAE CTF 2024 pwn (easy) "Command Recorder"

"Command Recorder"はstrcpyの仕様を題材とした問題です。
strcpyのmanには明確に

>  The strings src and dst may not overlap.

という記載があります。アルゴリズムの挙動（前から順にコピーする操作）を想像すると、たとえばコピー元とコピー先が1文字だけずれているようなケースでは、コピーされるデータが壊れることは容易にわかるでしょう。

この問題を作る上での難しいポイントは、このコアに対してどのように肉付けすると問題として成立するか、ということです。
コピー元とコピー先が重なるケースにおいて、コピーされるデータが壊れることは確かですが、それは別にメモリ破壊を引き起こすわけではなく、単に中のデータが壊れるだけです。
また、strcpyの引数が自然に重なるような、違和感のない実装にする必要があります。
こういったものを問題として成立する形に着地させるには、経験やテクニックが必要になってきますが、今回は「フラグを出力する機能自体はプログラムに元から備わってるものとし、データが壊れるだけでそれを利用できるようになる」という設定とすることで解決しました。
骨がある問題を作る場合には、複数のバッファを用意し、引数同士が重複しない、問題のないケースを混ぜ込むことによって、より発見を難しくするべきなのですが、初心者向けの問題であることを考慮して、仕様違反しているケースしか発生しないようにしました。

これ以上の詳細は[公式writeup](https://gmo-cybersecurity.com/blog/ierae-ctf-2024-writeup-pwn/#c)を見るといいでしょう。

#### 問題例1-b-2: Burning CTF 2016 pwn (medium) "Ninja no Aikotoba"

この問題ではstrcmpの仕様を題材にしました。

```
int is_matched(char *a, char *b, size_t n) {
    int result;
    char save = b[n];


    b[n] = '\0';
    result = strcmp(a, b);
    b[n] = save;
    if (result == -1 || result == 1) return 0;
    return 1;
}
```

C言語のstrcmpは、PHPのそれとは異なり、2文字列が異なる場合において、戻り値が-1や1になることは保証されていません。
つまり、strcmpが-1以外の負数や1以外の正数を返した場合、`is_matched`は2文字列は同一であるとみなしてしまいます。

この問題もこれをコアにした上でどう肉付けするかというのが難しいのですが、この部分は説明を割愛します。
気になる方は[ソースコード](https://www.dropbox.com/scl/fo/1xhr256gu6xbrluz5kb7i/AHKZw6-toIDmndi-Se-7tWA?rlkey=wkbyj2zbz9woyzbyrdesw8fih&e=2&dl=0)や、[kusanoさんのwriteup](https://qiita.com/kusano_k/items/ef75d4e1a3cc663cd03c#142-ninja-no-aikotoba)、[bonoさんのwriteup](https://bono-ipad.github.io/burningctf2015_writeup.html#142-%E6%94%BB%E6%92%83%E8%A1%93-ninja-no-aikotoba-300%E7%82%B9)などを見るといいでしょう。

### パターン1-c: 非自明な仕様

ライブラリやフレームワークには、非自明な仕様があり、先入観に囚われると躓いてしまうことがままあります。そういったものを題材とするのがこのパターンです。
「ありがちなミス」と分類としては同じであるとみなすこともできるかもしれませんが、極めて稀なケースでのみ踏んでしまう仕様など、必ずしも"ありがち"とは言えないようなものもあります。

#### 問題例1-c-1: CODE BLUE CTF 2017 pwn (medium) "nonamestill"

以下はURLエンコードされた文字列をインプレイスにデコードする関数ですが、脆弱性があります：

```c
char *cgiDecodeString (char *text) {
  char *cp, *xp;

  for (cp=text,xp=text; *cp; cp++) {
    if (*cp == '%') {
      if (strchr("0123456789ABCDEFabcdef", *(cp+1))
          && strchr("0123456789ABCDEFabcdef", *(cp+2))) {
        if (islower(*(cp+1)))
          *(cp+1) = toupper(*(cp+1));
        if (islower(*(cp+2)))
          *(cp+2) = toupper(*(cp+2));
        *(xp) = (*(cp+1) >= 'A' ? *(cp+1) - 'A' + 10 : *(cp+1) - '0' ) * 16
          + (*(cp+2) >= 'A' ? *(cp+2) - 'A' + 10 : *(cp+2) - '0');
        xp++;cp+=2;
      }
    } else {
      *(xp++) = *cp;
    }
  }

  memset(xp, 0, cp-xp);
  return text;
```

一目で脆弱性を見つけられたでしょうか。一見問題ないように見えるかもしれません。
strchrのmanを読んでみると、

> if c is specified as '\0', these functions return a pointer to the terminator.

という記述があります。上記の関数を見てみると、strchrの`c`に当たる引数は、入力から与えられるものであるため、攻撃者がコントロールできます。
たとえば意図的に、`"%A\0"`のような文字列を与えた場合、`c`を`\0`としてstrchrを呼び出すことができ、その戻り値はNULLにはなりません。よってバッファーオーバーラン、ひいてはバッファーオーバーフローが発生します。

## 2. 解法をコアとする場合

現実のセキュリティ診断とCTFが乖離しがちなポイントとして、CTFにおいては脆弱性を発見するだけで終わりではないという点が挙げられます。
「脆弱性は自明だが悪用が難しく、悪用方法を深く考える必要がある」という状況は、CTF（あるいは、セキュリティ診断を超えた現実のexploit）特有です。
このような問題は往々にしてパズル要素が強いということもあり、CTFらしい問題と言っても過言ではないでしょう。

解法をコアとする場合には、それ以外の要素が不必要に面倒にならないように気をつけるべきです。
悪用方法を考えることが本質であるにも関わらず、無駄に大きなソースコードを与えてそれを隠すような行為は、本質をぼかして問題を面白くなくしてしまうことが多いです。また、ソースコードが大きいと、想定外解法を生みやすくなるため、そういったミスを防ぐ上でも、本質以外の部分はシンプルにすることが望ましいでしょう（とはいえ、難易度調整のためにこの注意を無視することは上級者でもよくあります...）。

また、想定する悪用方法やテクニックが非常に新規性が高いものの場合、それにたどり着くためのヒントや、可能性を減らしてあげる工夫も、可能であればほしいところです。
そのようなものがなく、とても24時間や48時間で思いつけるとは思えないような想定解法の場合、単に自身の研究結果を発表しているだけの独りよがりな問題になる可能性があることに注意すべきでしょう（とはいえ、発見したテクニックを一度は出題しないことには、典型として使うことはできないため、一度は目をつぶるべき、と考えることもできるとは思います）。

また反対に、「初心者が解けるよう、難しいテクニックを使わないようにする」というのも、解法をコアとするケースの1つであると言えます。
その意味では、前述した"This is warmup"や"Watch Cats"などの問題は、解法にも注目していると言えます。

### パターン2-a: 挙動や性質を知る

カーネルやライブラリ関数などの挙動や性質を観察することで、非自明な悪用方法を思いつけるケースがあります。
たとえばheap exploit系の問題も、glibc mallocの挙動や性質を理解する必要があるという点で、これに分類していいかもしれません。

#### 問題例2-a-1: WCTF 2017 pwn (hard) "Under Debugging Revenge"

参加するチーム同士で問題を出し合う招待制CTF "WCTF"で出題したこの問題は、mmapの挙動を題材とした問題です。
脆弱性の部分は割愛すると、本質的にはmmapで作成したメモリ領域上で、バッファーオーバーフローを引き起こすことができます。
ここで、「mmapで作成されたメモリ領域がどこに作られるのか」というのを実験などによって知ることによって、どのように悪用できるかを思いつくことができます。

以降の詳細も省略しますが、ld.soが作る無名のメモリ領域を上書きすることで、[HITCON CTF 2015 Blinkroot](https://ddaa.tw/hitcon_pwn_200_blinkroot.html)と同様に、link\_mapを破壊し、任意のコード実行につなげることができます。

### パターン2-b: ソースコードを読んで見つける

ソースコード（あるいは仕様書）を読んで何かを見つけることで、非自明な、あるいは、新規性のある悪用方法を思いつく場合があります。
pwnに限らず、たとえばwebでは[Prototype Pollutionのガジェットを探してそれを出題する](https://blog.arkark.dev/2023/09/21/seccon-quals/#Solver-6)ようなものがこのパターンとして挙げられます。

#### 問題例2-b-1: IERAE CTF 2024 pwn (hard) "New Under The Sun"

"New Under The Sun"はIERAE CTFのボス問として作成した問題です。
ソースコードは以下の通り、非常にシンプルです：

```c
// gcc chal.c -o chal -O3 -no-pie

#include <stdio.h>
#include <stdlib.h>

_Thread_local char buf[16];

int main() {
  long long int idxs[11] = {};
  long long int vals[11] = {};

  for (int i=0; i<11; i++) {
    scanf("%lld%lld", &idxs[i], &vals[i]);
    if (idxs[i] < 0 || 0x3000 <= idxs[i]) continue; // Idiot
    buf[idxs[i]] = vals[i];
  }

  exit(0);
}
```

しかし、このコードが作成されたのは最終段階であり、この問題の作問はglibcのソースコードを読むところからスタートしています（つまり、その時点ではこのようにシンプルなコードになることも全く想像していませんでした）。
私はglibcを読むのが趣味であるため、たまに読んでいるのですが、\_\_libc\_start\_mainの以下の部分がずっと気になっていました：

```c
  not_first_call = setjmp ((struct __jmp_buf_tag *) unwind_buf.cancel_jmp_buf);
  DIAG_POP_NEEDS_COMMENT;
  if (__glibc_likely (! not_first_call))
    {
      struct pthread *self = THREAD_SELF;

      /* Store old info.  */
      unwind_buf.priv.data.prev = THREAD_GETMEM (self, cleanup_jmp_buf);
      unwind_buf.priv.data.cleanup = THREAD_GETMEM (self, cleanup);

      /* Store the new cleanup handler info.  */
      THREAD_SETMEM (self, cleanup_jmp_buf, &unwind_buf);

      /* Run the program.  */
      result = main (argc, argv, __environ MAIN_AUXVEC_PARAM);
```

これがどのように使われているかを細かく追っていった結果、TLS領域の書き換えによって悪用できることがわかり、最終的には上記のコードが作成されました。

### パターン2-c: 実験して見つける

ときとして、対象とするライブラリ関数やシステムコールなどに対して実験を行うことで、悪用の方法を思いつけることがあります。これは、「ソースコードを読んで見つける」の代替だと考えることも可能でしょう。理想的には、すべてのソースコードを読み、対象の動きを完全に理解すれば、何かを思いつくことができるかもしれませんが、場合によっては対象のソースコードが膨大であり、そんな余裕がないことはあります。そのような場合に、ブラックボックス的に外から実験・観察することにより、何かを発見できる場合があります。

#### 問題例2-c-1: IERAE CTF 2024 pwn (medium) "Intel CET Bypass Challenge"

その名の通り、ROPの緩和機構であるIntel CETを回避する問題です。
「Intel CETはsingalを使うことで回避できる」というシンプルな発見を、そのまま問題にしています。
詳細については[解説記事](https://gmo-cybersecurity.com/blog/intel-cet-bypass-on-linux-userland/)を読むとよいでしょう。

実際には実装するまで実験は一切しておらず、突然頭に降ってきた（何故か検証もせずに、実装するまでずっとできると信じていた）だけなのですが、本来であれば実験して発見するような問題かなと思います。

### パターン2-d: 制約から考える

私が難しい問題を作る上で、たまに使うパターンが、制約から考えるです。
つまり、作問を開始した時点では、自分も答えを知らない状態で、何らかの制限を課したプログラムを1から攻略することで、解法を思いつき、出題するということです。
これは往々にして、単純にexploit不可能という結論にいたり失敗することが多いですが、うまく行くと難しい問題が誕生します。

#### 問題例2-d-1: CODE BLUE CTF 2018 Finals pwn (medium) "fliqpy"

以下のコードをとりあえず書いてから、exploitできるかを検証しました：

```c
/*
 *  gcc fliqpy.c -o fliqpy 
 */

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

__attribute__((constructor))
void set_timeout(void) {
  alarm(150);
}

int main(void) {
  unsigned char *addr;
  unsigned long long int ord;

  puts("You know, actually soft error is a common phenomenon.");
  puts("It occurs with non-negligible probability, not astronomical odds.");
  puts("So this time let's see if you can exploit this only with one bit flip!");

  printf("Enter the address you want to flip: ");
  scanf("%p", &addr);
  printf("Which bit do you want to flip?: ");
  scanf("%llu", &ord);
  if (ord >= 8) {
    puts("Idiot");
    exit(0);
  }
 
  *addr ^= 1 << ord;
  exit(0);
}
```

結果、できました。

## 3. 対象をコアとする場合

脆弱性・解法をコアとしない場合、対象の新規性をコアとすることがあります。
これをコアとする問題では、非自明な脆弱性を見つけたり、パズル的な解法を思いついたりといった面白さが欠落していることもあります。
しかし、これまでCTFで（あるいはひょっとすると現実世界でも）扱われてこなかったプログラムやライブラリを対象として攻撃を行うということは、セキュリティの研究としての意義は大いにあるわけなので、あながち意味がないとは言い切れません。
すなわち、この手の問題では、もっぱら教育的な要素が強くなります。

たとえば、過去にはLinux kernelやブラウザ・JavaScirptエンジンを攻撃する問題はこれに分類されていました（今ではすでに典型と化し、様々な発展的な問題が出題されていますが）。
あるいは、今でも、Windowsのuserland・kernelの問題はこれに分類されることもあるでしょう。

#### 問題例3: Asian Cyber Security Challenge 2024 pwn (easy) "fleeda"

fridaでアタッチされているプログラムをexploitする問題です。
これまで、QEMU escapeやpintoolを題材とした問題は過去に出題されていましたが、fridaを題材としたものは見たことがないため作成しました。

# まとめ

いかがでしたか？
CTFの作問は、ひょっとするとCTFに参加して難しい問題を解く以上に難しいな、と思います。
CTFの作問で悩んでいる人というのは少ない気もしますが、そのような悩みを抱えている人にとって、この記事が少しでも役に立つことを願っています。
本当は肉付けするときのテクニックや、その他作問時に気をつけるポイントなども扱いたかったのですが、時間がありませんでした（sorry）。

明日というか今日の記事はSatoooonさんの「Tanuki Udonを供養する」です。
想定外解法しか知らないので楽しみですね。
