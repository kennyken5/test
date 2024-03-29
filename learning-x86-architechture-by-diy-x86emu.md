# 自作エミュレータで学ぶＸ８６アーキテクチャ

## まえがきから

本筋とは関係なかったが、ＭＤ５０のルールの４１、
最初の行は見出しレベル１でなければならない
が、指摘されてしまった。

以上、あとから挿入したもの。

書籍自作エミュレータで学ぶＸ８６アーキテクチャを読む

全198頁
全４章
chap.1 Ｃ言語とアセンブリ言語（と機械語との諸関係とは？）
chap.2 ポインタとアセンブリ言語（：Ｃのポインタをアセンブリ言語から眺める）
chap.3 ＣＰＵがプログラムを実行する仕組み（：本書の中心をなす。ＨＤＤに書かれたプログラムがどう読み込まれるか）
chap.4 BIOSの仕組みと実機起動

導入

問う。エミュレータ、コンピュータ分野のそれとは何か？
答える。まずそれはソフトウェアのひとつである。
他のハードウェアやソフトウェアを模倣する別のソフトウェアである。
模倣でなけれなエミュではなない。
例えばオリジナルのコピーはエミュではない。

問う。本書の扱うエミュはどのようなものか？
答える。まず、本書が扱わない種類のエミュを説く。
例えば、VirtualBox。いま主流の仮想化方式によるエミュ。
現実ハードウェアのCPUを間借りするものであり、
CPUの動作をソフトウェアでエミュレートするものではないため、
このＣＰＵ間借り方式では、
x86_CPUの上でARM_CPU向けのソフトウェアを動かすことはできない。
しかし、同じx86_CPU向けのソフトウェアであれは高速に動かせる。
ＣＰＵ間借り方式は速度が出せるため実用には適しているが、エミュの学習には不向きである。
したがって、本書が扱うのはＣＰＵ動作をソフトウェアで模倣し、
その上で別のソフトウェアを実行委するＣＰＵエミュである。
ｍ２のｍ68kもそうだね。左はモトローラＣＰＵであり、
本書が扱うのはインテルＣＰＵ。のはず。

問う。ＣＰＵエミュを作る意義は何か？
答える。まず、現実にＣＰＵ制作するのは電子工作である。エミュならはんだごては不要である。
次に、エミュレータ制作を通じてコンピュータの内側を学習できる。
ひとつ。コンピュータを構成する各要素を理解。CPU記憶装置入出力装置など。
ひとつ。機械語とアセンブリ言語の理解。
ひとつ。エミュ制作のためのプログラミング言語の理解。
これらは個別に学ぶこともできるが、ＣＰＵエミュを作るならひとセットで行ける。
コンピュータの低水準な部分を一通りさらえるのである。

問う。本書が扱うＣＰＵの種類は何か？
答える。インテルの８０３８６である。いまのインテルコアの直系祖先であり、
その採用するｘ８６アーキテクチャは、いまのいんてるこあＣＰＵに上位互換。

８ビットである８０８０を１６ビットに拡張したのが８０８６である。
８０８６はＮＥＣのＰＣ９８に採用された。
姉妹品の８０８８はＩＢＭＰＣに採用された。
８０８６の後継が、８０２８６や８０３８６である。
上３つを総称してｘ８６アーキテクチャと呼ぶ。

１９８５年に８０３８６が発売された。ｘ８６アーキテクチャで初の３２ビットＣＰＵである。
３２ビットなので、扱えるメモリ空間は４ＧＢに拡張された。
２００４年にペンティアム４の第３世代インテル６４アーキテクチャまで
パソコン用ＣＰＵデファクトスタンダードとなった。
（テル６４は、ｘ８６－６４とかＡＭＤ６４と呼ばれる）

## chap.1 Ｃ言語とアセンブリ言語
はじめに／言語ｃで書かれたプログラムがアセンブリ言語で書かれたプログラムに変換される様を説明する。
また、アセンブリ言語と機械語の関係性についても説明する。

### １－１：Ｃ言語から機械語へ

我々がふつうに書くＣ言語プログラムのファイル形式はテキストファイルである。
ＣＰＵはファイルをバイナリファイルをして読む。

ＣＰＵが解釈できるのはもっと原始的な命令、つまり機械語の列だけを解釈できる。
反対に言うと、ＣＰＵが解釈できる命令こそ機械語と呼べるものである。

基板上の電子回路では、電圧の高い低いで０か１が記録されている（ふつう1.35Vと０V）。
同様に、ＨＤでは磁石のＳ極Ｎ極で、ＤＲＡＭではコンデンサに電荷が溜まっているか否かで記録されている。

### １－２：機械語とアセンブリ言語

ひとにとって機械語は非常に読みづらい。その困難を解決すべく、アセンブリ言語が作られた。
アセンブリ言語は機械語と一対一対応しているが、機械語よりも多少読みやすくなっている。
（その中間にASCIIコードがあるような気がする。コードにはASCIIコードと異なり非８ビットのもの、例えばテレタイプのものなどもあるが）

リスト１．１ Ｃ言語のプログラム
void func(void) {
    int val = 0;
    val++;
}

リスト１．2　上を機械語に変換したもの
55,89,e5,83,ec,10,c7,45,fc,00,00,00,00,ff,45,fc,c9,c3

リスト１．4　上にアセンブラ言語に変換したものを加えたもの
push    ebp 55,
move    ebp,    esp 89,e5,
sub     esp,byte +0x10  83,ec,10,
mov    dword [ebp-0x4],0x0  c7,45,fc,00,00,00,00,
inc      dword [ebp-0x4] ff,45,fc,
leave        c9,
ret           c3,

前述したように、アセンブラ言語で書いたプログラムの文は機械語と一対一対応する。
しかし、Ｃ言語のプログラムは機械語と一対一対応しない。以下、例として、int型変数valに対して、

val++;

または、

val　＋＝　1;

と書いたとき、これをＣにコンパイルすると、

inc [ebp-4]

となるか、

add[epb-4],1

となるか、それとも全く異なるものとなるかもしれない。
実行されていること自体は同じであり、変数valに1を加える、である。

ことば：
Ｃ言語プログラムを機械語に変換することをコンパイルと言う。
アセンブリ言語プログラムを機械語に変換することはアセンブルと呼び、コンパイルと呼ばない。

最後にアセンブルの例外について記す。

ひとつの機械語命令が複数のアセンブリ言語命令に対応する例。
0x90 は nop と xchg eax,eax に対応する。
nop は no operation の意味で、何もせず時間を消費する命令である。
xchg eax,eax は、 eax と eax を入れ替える命令である。
xchgはexchange、eaxはレジスタである。
結果的にnopと同じ内容となる。

複数の機械語命令がひとつのアセンブリ言語命令に対応する例。
機械語の「0x40」と「0xff 0xc0」とがアセンブラ言語の「inc eax」に対応する。
両者はレジスタ名の指定のやり方が異なる。

オペコード「0x40」はレジスタeaxの指定を含むが、
オペコード「0xff」はレジスタeaxの指定を含まず、単にinc命令であるだけなので、レジスタの指定が必要である。
後続の1バイト0Xc0（の中でも、6-7ビット目11および0-2ビット目000）により、inc命令の操作対象がeaxであることを示す。
（実は0xffはincまたはdecを表す。そのどちらかかの指定は、ModR/Mバイト0xc0の3-5ビット目で決まる。後述）

おまけ
x86はCISCアーキテクチャであり、各命令のバイト数が異なっていたり、高度な計算をまとめて行う命令があったりするが、
RISCアーキテクチャの特徴を持つARMでは、多くの命令が4バイトの固定長である。

原著の17ページには、リスト１．１ Ｃ言語のプログラムをARM用gccでコンパイルした例が載っている。

00000000 <func>:
0:e52db004　　　Push{fp};(str fp,[sp,#-4]!)
以下、略

### １－３：機械語に飛び込む
演習。用意されたサンプルファイルを用いて逆アセンブルを行う。

Ｃ言語で書かれたプログラムのテキストファイル「casm-c-sample.c」の内容は以下。

    void func(void) {
    int val = 0;
    val++;
    }

　
このファイルをgccを用いてコンパイルして機械語の実行ファイルに変換し、
さらにそれを逆アセンブルする（命令ndiasemによって）。

> ndisasm -b 32 casm-c-sample.bin

ここで「-b 32」とし、32ビットモードのプログラムと指定する。
この指定を抜いて実行すると、16ビットモードで解釈されてしまう。

### １－４：アセンブリ言語を少し詳しく

アセの文法
オペコード オペランド1,オペランド2
オペコとオペラを合わせたものをニーモックと呼ぶ。
Ｃ言語に対応させると、オペコは関数名、オペラは実引数、ニーモは適当な対応物なし。

### １－５：基本のmov命令

move 移動先,移動元

先に対し、元のデータをコピーする。一度にmovできるデータは1,2,4バイトに限られる。

1.mov ebd,esp

レジスタebdの中身をレジスタespへmov（コピー）する。
この２つのレジスタは任意の32ビットの値を記録する。

espはスタックポインタ。常にスタックの最新の読み書き位置を保持する。
ここではスタックとはローカル変数を置くためのデータ領域と理解せよ。

2.mov dword [ebp-0x4],0x0

コピー元が0x0で、これは番地ではなく即値またはリテラルと呼び、この場合は数値0である。
コピー先がdword [ebp-0x4]である。

ebp-0x4はメモリ番地
[]とは囲みの内側が数値ではなくメモリ番地を意味づけするためのもの
dwordは領域の大きさを指定するもの(32ビットである)

各メモリ番地には１バイト＝８ビットのデータが記憶できる。

なお、ここで示される変数valの位置[ebp-0x4]とはＣの規格によるものではなく、今回のコンパイルでたまたま選ばれただけ。

アセンブリ言語では、ある領域を示すために２種類の情報を要する。先頭のメモリ番地とその領域の大きさである。

### １－６：インクリメント専用のinc命令

inc インクリメント対象

例のプログラムでは、

inc dword[ebp-0x4]

となっている。変数valに1だけ加算する。

さて、1-2で次のように書いた。

> 機械語の「0x40」と「0xff 0xc0」とがアセンブラ言語の「inc eax」に対応する。
> inc      dword [ebp-0x4] ff,45,fc,

アセンブリ言語による「inc dword [ebp-0x4]」を機械語にすると「ff,45,fc」と変換されている。

また、1-5での説明から、機械語45,fcは「ebp-0x4が」にあたると分かる。
なので、機械語のffがインクリとなる。

機械語命令の構成について述べる。

【inc dword [ebp-0x4] 】こと【ff,45,fc】は、

　　プレフィクス　0～4バイト
ff　　オペコード　1,2,3バイト
45　ModR/M　0,1バイト
　　SIB　　0,1バイト
fc　ディスプレイスメント　0,1,2,4バイト
　　イミディエイト　　0,1,2,4バイト

と図示できる。

オペコードffはその命令がinc命令あるいはdec命令であることを示す。
ModR/Mはメモリ領域の一の決め方（位置自体でなく、位置の決め方）を指示するもので、
ここでの45とは「epb+8ビットディスプレイスメント」で示される番地のメモリ領域を表す。
最後のfcはModR/Mの8ビットディスプレイスメントを示す。
ディスプレイスメントとは位置の差分のこと。

なお、i386の32ビットモードにおいてはdword指定がデフォルトのである。
ゆえに、機械語にはdword指定に相当する記述はない。
アセンブリ言語ではサイズ指定が必須なので記述されている。

DWORDは4バイト
WORDは2バイト
BYTEは1バイト

修飾子と呼ぶ。

### １－７：16進数入門

（略）

### １－８：2の補数入門

コンピュータ分野では、二進数のとき、負の数を表す際に、「補数」を用いるのがならい。
0d41(＝十進数デシマルによる四十一）は、0b00101001（＝二進数による十進の四十一）。
0b1,0000,0000マイナス0b0010,1001は0b11010111．これをマイナス４１とする。冒頭の1はマイナスの意味。
当然、この数字は0d215とも読める。
（なお、全ビットを反転して1を加えるという計算だと速い）

cの規格ではこの「２の補数」の使用は必須ではないのだが、実際上ではほとんど100％である。

## chap.2 ポインタとアセンブリ言語

### ２－１：レジスタ

以下のＣ言語のプログラム

int i,sum=0;
for(i=0;i<10;i++){
    sum+=i;
}

0から10未満までの自然数を足し合わせる

ハンドアセンブルすると（一例）、

 mov eax,0 ;;;; int i,sum=0 
 mov ecx,eax ;;;; i=0
 jmp loop_end ;;;; "loop_end"に飛ぶ
loop:
 add eax,ecx ;;;; sum+=i;
 inc ecx ;;;; i++
loop_end:
 cmp ecx,10 ;;;; i<10
 jl loop ;;;　が真なら"loop"に飛ぶ

レジスタecxが変数iに、レジスタeaxが変数sumに、対応する。

レジスタの大きさ、数、名前、役割などはＣＰＵに固有なため、
Ｃ言語プログラムは同じコードを異なるＣＰＵで動かせるが、
アセンブリ言語プログラムはほぼ無理である。



### ２－２：メモリ

386には32ビット幅の汎用レジスタが8つある。
旧a,b,c,d(アキュムレジスタ、ベースレジスタ、カウントレジスタ、データレジスタ)だった、
eax、ebx、ecx、edxの４つと…（eは拡張extend）

インデックスレジスタ
esi ソースインデックス
edi ディスティネーションインデックス

特殊レジスタ
esp スタックポインタ
ebp　ベースポインタ

（以上の8つのレジスタ説明はキクチが他所から引用したので、正しいかどうか不明）

◆

メモリの0x7c00番地から0x7c03番地までの4バイトのデータを、
メモリの0x7a00番地から0x7a03番地までの領域にコピーする。

mov eax,[0x7c00]
mov [0x7a00],eax

あるいは、

mov ebx,0x7a00
mov eax,[ebx+0x200]
mov [ebx],eax

さて、c言語の未resister指定な変数は、
プログラマの視点からはメモリに置かれる。
そのため、未resister指定な変数は、
必ずその変数が置かれたメモリ番地を持つ。

&演算子が適用された変数は、
最適化によって決してレジスタに置かれることは起きない。
なぜならば、レジスタはＣＰＵ内にあるため、
レジスタはメモリ番号を決して持ちえないのである。

また、resister宣言された変数へ&演算子を適用しようとすると、
実際にレジスタに置かれるか否かに関わらず、コンパイルエラーとなる。

◆lea命令

void func(void){
    int val;
    int *ptr=&val;
    *ptr=41;
}

以上のアセンブル結果（ndisasmの出力を修正し、対応するC言語のソースコードを追記）は以下

########;void fund(void){
push ebp
mov ebp,esp
sub esp,16

########; int val;
########; int *ptr=&val;
lea eax,[ebp-8]　　　（１）
mov [ebp-4],eax　　　（２）

########; int *ptr=41;
mov eax,[ebp-4]　　　（３）
mov dword[eax],41　　　（４）
}

leave
ret

pushとleaveは関数の入り口と出口での定型的な処理をするもの。
（１）lea命令により、変数val([ebp-8])の番地がレジスタeaxに格納される。
（２）レジスタeaxの値が変数ptr([ebp-4])に格納される。
（３）変数ptr([ebx-4])の値をレジスタeaxに読み出し、
（４）そのレジスタeaxの値で示される4バイトのメモリ領域に41を書き込む。
こうして最終的に変数val([ebp-8])に41が書き込まれる。

アセンブリ言語の世界では、レジスタは名前で呼び、メモリは番地でその場所を表す。

c言語の世界ではメモリに名前を付ける機能があり、それを「変数」と称する。
ｃ言語の「変数」とは、つまり特定のメモリ番地に“val"のようにひとに分かりやすい名前を付ける。

機械語の世界では、レジスタに名前はなく番号が振られている。
レジスタeaxは0、ecxは1、というふうに。

機械語はレジスタもメモリ番地も数値で指定する。
アセンブリ言語ではレジスタは名前で、メモリ番地は数値で指定する。
Ｃ言語はレジスタは使えず、メモリ番地は名前で指定する

余談。int型のような複数ナイトにまたがる変数に値を格納するとき、
どんなバイトの順序にするかはＣＰＵによって異なる。
本書の扱うintelのＣＰＵは小さい桁が小さい番地に配置される「リトル・エンディアン」。

変数ptrに0x00007bf0という4バイトの値が格納されたとき、
メモリ上では、0xf0、0x7b、0x00、0x00、と並ぶ。

なお、ネット上を流れるデータでは「ビック・エンディアン」に統一することに決まっている。
0x00、0x00、0x7b、0xf0、である。


### ２－３：初めてのエミュレータ

（あとから挿入　本文では、main.cの全文解説は行われていないので、フォルダtolset_p86\emu2.3を見よ


まず、エミュ本体を表す構造体を定義する。
次に、エミュ構造体を生成して初期化し、ファイルから機械語プログラムを読み込む処理を書いてゆく。
そして、エミュの核心である機械語を実行する部分を作ってゆく。

i386の32ビットモード用の機械語プログラムが書かれたファイルを受け取ると、
それを先頭から順に実行してゆくようなものを作る。

リスト2.5：main.c
typedef struct{
    /* 汎用レジスタ  */
    uint32_t registers[REGISTERS_COUNT];

  /* EFLAGSレジスタ  */
   uint32_t eflags;

   /* メモリ（バイト列）  */
   uint8_t memory;

   /* プログラムカウンタ  */
   unit_t eip;
}　Emulator;


エミュ構造体を生成して初期化し、ファイルから機械語プログラムを読み込む処理するもの。
コマンド名はpx86とする。コマンドライン引数に機械語プログラムが格納されたファイルを指定する仕様とした。
そこで、main関数の頭でまずコマンドライン引数がひとつ指定されていることを確認する。
続いてcreate_emuでエミュ構造体を作成して初期化。
最後に機械語のファイルを開き、freadでemu->memoryに読み取る。

リスト２.6：main.c
Emulator* create_emu(size_t size, unin32_t eip, uint32_t esp)
{
    Emulator* emu=malloc(sizeof(Emulator));
    emu->memory=malloc(size);

    /* 汎用レジスタの初期値をすべて0にする */
    memset(emu->registers,0,sizeof(emu->registers));

    /* レジスタの値初期値を指定されたものとする */
    emu->eip=eip;
    emu->resisters[ESP]=esp;

    return emu;   
}

/* エミュレータを破壊する */
void destory_emu(Emulator* emu)
{
    free(emu->memory);
    free(emu);
}

int main(int argc,char* argv[])
{
    FILE*=binary;
    Emulatoer* emu;

    if(argc !=2){
        printf("usage:px86 filename\n");
        return 1;
    }
/* EIPが0、ESPが0X7C00の状態のエミュレータを作る */
emu=create_emu(MEMORY_SIZE,0x0000.0x7c00);

binary=forpen(argv[1],"rb");
if(binary==NULL){
    printf("%sファイルが開けません￥n",argv[1]);
    return１；
}

/*　機械語ファイルを読み込む　最大512バイト　*/
fread(emu->memory,1,0x200,binary);
fclose(binary);

destory_emu(emu);
return　0;
}

main.cのソースコード解説部分はここでパスすることにする。
このエミュに実装した命令は、即値をレジスタに書き込むmovとショートジャンプのjmpのみである。
ゆえに、以下のようなファイルhellowworld.asmのプログラムを起動できる


hellowworld.asm
BIT 32
start:
    mov eax,41
    jmp short start

tolset_p86\pasm-helloworldにある!cons_nt.batを実行せよ。
そして現れるターミナルにて、makeを実行せよ。
すると、helloworld.binが作られる。

次にエミュをビルドする。
tolset_p86\emu2.3の!cons_nt.batを実行せよ。
そして現れるターミナルにて、makeを実行せよ。
すると、px86.exeが作られる。

この状態で、
px86.exe ..\pasm-helloworld\helloworld.bin

レジスタeaxの値が29なら成功　ｏｋ

### ２－４：ポインタの復習

リスト2-10
void func(void){
    int val;
    int *ptr=&val;
    *ptr=41;
}

の変数funcを実行すると、変数valに値41が書き込まれます。
このプログラム中に変数valへの代入文は不在であるけれども、
ポインタ変数ptrを介して間接的に書き込まれるつくり。

　valは整数を格納するint型。
　ptrはint型変数を示すポインタを格納するint*型。
　ptrの初期値が&val（変数valの番地）なので、
　４行目の段階でptrはvalを指し示す状態になっているので、
　*ptr=41を実行すれば、valに41が代入される。
　なお、ここの*は演算子で、ポインタが指し示している変数を間接参照するもの
　間接参照や参照外し（dereference）という。

次にもう少し実用的な例を見よう。

ｃ言語では、複数の値を戻り値として直接返す方法がない。
整数の割り算を行う関数mydivする。
商quotientと余remainderを同時に呼び出し側に返すため、
２つのポインタ引数quotとremを取っている。
あるいは複数の変数をまとめた構造体を定義して戻り値に使うか。


### ２－５：ポインタに飛び込む

ポインタはメモリ番地と。指し示す領域の大きさを組み合わせた概念である。
ある型をＴとしたとき、T*はＴ型へのポインタを表す型となり、
指し示す先がＴ型であることを示す。
Ｔ型の大きさはsizeof(T)であるり、Ｔによって変わる。
しかし、ポインタ変数は値としてメモリ番地を格納するため、
Ｔが何型であってもポイント変数自身の大きさは同じである。

ポインタをつかったＣのプログラムをアセンブリ言語に変換すると、
Ｃのポインタ変数とｉｎｔ型変数とは区別がないことが見て取れる。
アセンブラの視点では、どちらもビット列を格納したメモリ領域にすぎない。
ただ一点、そのメモリ領域から取り出した領域を
さらに[]で囲んで使うのがポインタ変数に特徴的な部分といえるが、
それもＣコンパイラが生み出した違いにすぎない。


### ２－６：構造体とポインタ

構造体とは、複数の変数をまとめて新しい型（ユーザー定義型）を作る機能。


間接参照、参照外し、dereferenceを復習。

リスト2.21
Emulator emu;
Emulator *p=&emu;
(*p).eflags＝０;
p->eip＝０;

なお　*p.eflags　と書いたら、　*(p.eflags)の意味になり、コンパイルエラーをみちびく。

「->」は演算子であり、
「X->Y」は「(*X).Y」と同じ意味。ただし、構造体のポインタである。

.は構造体メンバ演算子。メンバとは、構造体を構成する要素。変数やポインタ変数。


### ２－７：不完全型とポインタ

定義は宣言の一種だが、特にメモリの割り当てを伴うものを定義と呼ぶ傾向あり。

ある型の変数定義には、その変数の大きさを知ることが必要。
基本型ならすでに定められているが、構造体などユーザー定義型の場合は変数定義の場合に型の詳細を知らねばならない。

構造体の前方宣言（forward declaration）はできる。

Ｃ言語の前方宣言例
int elements[]; /* int型の配列変数の前方宣言 */
void foo(int); /* int型の引数を受け取る関数の前方宣言 */
struct user_type; /* 構造体の前方宣言 */


関数の前方宣言はまた関数プロトタイプでもある。
コンパイラはソースコード中に出現したこれらの宣言を処理した後、
プログラマに以降の部分でelements, foo, user_typeの実体の使用を許可する。
ただし、宣言はあくまでプログラムにおける各シンボルの意味（役割）を示すだけである。
また、要素数が明らかでない静的配列や、メンバーが分からない構造体は不完全な型 (incomplete type) であり、
sizeof演算子を適用したり、メンバーにアクセスしたりすることはできない。
最終的にプログラマは（例えば以下のような）宣言した実体のための定義をどこかで提供しなければならない。

int elements[10];
void foo(int x) { printf("%d\n", x); }
struct user_type { int a; double t; };

特に前方宣言をインターフェイスとしてヘッダーファイルに分離し、
実体をソースファイルに記述することで、個々のプログラム部品の実装を隠蔽して独立性を高めることができるようになる。

struct List *plist;

のように、ポインタ型の変数定義ならコンパイルエラーにならない（ポインタ型はだいたいint型だから、でよいかな？）

ライブラリの実装詳細をユーザから隠すために必要な行為は以下。
構造体の中身を読み書きするには必ず関数を使うとし、
それらの関数に構造体のポインタを渡すようにする。
すると、構造体の詳細が必要なのはそれらの関数内部に限定され、
ライブラリのユーザーには構造体の完全な宣言を公開せずに済む。
すると、構造体の実装を変更しても、その実装をユーザコードを再コンパイル必要となり、
ユーザーコードからライブラリの実装詳細が隠蔽される。これをカプセル化と呼び、
オブジェクト指向プログラミングでよく現れる。

ヘッダファイルに前方宣言のみ書き、.cファイルの中でその構造体の完全な宣言を書く。
c++ではより強く求められる。plmplイディオムとして知られる。

void型も不完全型の一種である。もともとは引数や戻り値が存在しないことの明示するものであったが。
void*型は「指し示す型は不明だが、何かしらのメモリ位置を指し示すポインタ」である。万能ポインタ型と呼ぶか。
標準ライブラリ関数ではどんな型にでも適用できるようにｍvoidポインタを引数や戻り値にする関数が多数あるる。


mallocの戻り値をポインタ変数に代入したり、配列に０をコピーするのを、voidポインタを意識せず書ける。


### ２－８：関数ポインタ

アセンブリ言語でのサブルーチンに、引数と戻り値をつけ足したものがＣ言語の関数だといえる。


## chap.3 ＣＰＵがプログラムを実行する仕組み

ここまでの諸章で見てきたことは、ＡでなくＢである。
Ａ　コンピュータシステムとしてプログラム動作の初めから終わりまで
Ｂ　CPUが理解する機械語、アセンブリ言語とＣ言語の対応、アセンブリ言語によるメモリの読み書き
それゆえ、この章ではＡを見る。
ＨＤＤに書き込まれた機械語プログラムがＣＰＵによってメモリ上に読み出されてから実行されるというプログラム起動時の話から、
ＣＰＵが実際にプログラムを実行し出力を行うまでの一連の流れを説く。
その中でＣ言語におけるローカル変数の定義や関数呼び出しとスタックとの関係に触れる。
またＣＰＵが外界と相互作用するための入出力についても説明し、
コンピュータと人間がどのようにやり取りされるかを見る。


### ３－１：プログラムの配置

ＣＰＵは内蔵するプログラムカウンタ(eipレジスタ)が差すメモリ領域から機械語命令を読み取って解釈し、その命令に従って動作する。
ここでいうメモリとはメインメモリとも呼ばれる部品である。
引用元の書籍執筆時にはDDR3-SDRAMという規格の部品が主流であった（2022夏はDDR4や５。win10～11）。

CPUはメモリにおいてあるプログラムしか実行できない。
メインメモリは揮発性があるので、CPUに実行してもらいたいプログラムは
最初に不揮発性の記憶装置に書き込んでおき、適宜メインメモリに読み出すようにする。
このとき使われる不揮発性の記憶装置はHDDやSDDなどあり、
メインメモリ（主記憶装置）と対比させて、補助記憶装置と呼ぶ。
CPUから補助記憶装置へはI/Oを介する。

プログラムローダ(program loader)はその実行させたいプログラムをメモリに読み出す。
その後、プログラムローダは読みだしたプログラムの先頭へジャンプする。

このプログラムローダも機械語のプログラムであるから、
最初のプログラムローダはだれが読みだすのかという問題が生じる。

結論を言うと、それがBIOSである。
不揮発性の記憶装置に書き込まれていながら、
メモリと同列に接続されている。

一番簡単なのは、機械語プログラムそのままのバイト列を格納すること。
Raw Binaryと呼ばれ、MBR（マスター・ブート・レコード）という特殊な領域に書き込むのに用いる。
機械語そのままなので、メモリに呼び出して先頭にジャンプするだけでプログラムが実行される。

いまではほとんど使われなくなったのがCOMという形式。MSーDOSでよく使われていた。
ファイルの中身をメモリの0x100番地からの領域に配置するだけでよし。

windowsならＰＥ形式、MacならMac-O形式、UnixならEFL形式。
いずれも機械語のバイト列にヘッダを付与したファイル。
ヘッダ部には、機械語のバイト数や、各セクションのサイズなどの管理情報が含まれている。

リスト3.1(2.26再掲)

start:              ;　プログラム開始
    mov si,msg ;　siに文字列を設定し
    call puts      ;　サブルーチンを呼び出す
fin:
    hlt
    jmp fin       ; 永久ループ

puts: 　　　　　； 
    mov al,[si]     ; １文字読み込む
    inc si             ; siを１文字進める
    cmp al,0        ; 文字列の末尾
    je puts_end    ; に来たら終了
    mov ah,0x0e  ; 
    mov bx,15
    int 0x10         ; BIOSの機能を呼び出す　COM形式という前提なので
    jmp puts
puts_end:
    ret                ; サブルーチンから抜ける

msg:                 ; 
    db "hello world",0x0d,0x0a,0

このプログラムを実行すると、SIレジスタにmsgのメモリ番地を設定してからputsを呼び出す。
callはスタックの先頭にcall命令の次の命令が置いてあるメモリ番地をプッシュしてから、目的のサブルーチンにジャンプする。
進んだ先でretに出会うと、スタックの先頭からポップする。

アセンブリ言語世界でのサブルーチンに、引数と戻り値を付けたしたものをＣ言語の関数だといえるだろう。

リスト2.27  関数ポインタ

int inc(int v){
    return v+1;    
}
int (*ptr)(int)=&inc;

４行目、関数の型(プロトタイプ宣言の関数名以外)を示す部分が int(int)
関数ポインタであることを示す部分が(*)、
変数名を示す部分がptr
関数ポインタptrに初期値として&incを設定する。

int *ptr(int)だと、戻り値がint*と解釈され、
「intを受け取りintへのポインタを返す関数ptr」という意味になる。


リスト2.28　typedefをつかう

typedef int func_t(int);
func_t *ptr2=&inc;

まず。関数の型そのものにfunc_tと名付ける。
そして関数ポインタ型をfunc_t*と表す。
こうすると、普通の関数プロトタイプ宣言と同じ形で、
ptr2の定義もそのほかのポインタ変数と同じになり、わかりやすい。


リスト2.29

int (*arr1[2])(int)={&inc,NULL};
func_t *arr2[2]={&inc,NULL};

このarr1とarr2とは定義の形は異なるが、
同じ型と初期値を持つ配列。

リスト3.1(2.26再掲)のところまで戻る。
実行ファイルはメモリの特定の番地に配置せよ。
さもなくばメモリ番地を扱う命令は誤動作する。
それらは絶対ジャンプ命令やラベルを使う命令だ。

org疑似命令を使うと先頭番地は変更する。
orgはアセンブラソフトウェアへの命令であり、
ＣＰＵが実行するものはない。だから疑似命令を呼ぶ。

番地0x7c00：BIOSが補助記憶装置のMBR（マスターブートレコード）からプログラムを読み出し、番地0x7c00に配置する。標準
詳しくは後の章で行う。

メモリはひとつしかなく、特定の番地を持つメモリ領域はただひとつしかないが、
ＣＰＵにはアドレス変換という機能によりマルチタスクに対応する。
プリグラムからみたメモリ番地（論理アドレス）と
ＣＰＵがアクセスするメモリ番地（物理アドレス）を変換することで競合を避ける仕組みである。
あるプログラムは物理アドレスの0x100000から配置し、
あるプログラムは物理アドレスの0x200000から配置するが、
プログラムから見るとどちらも0x400000から配置されている、という状況が作れる。

アドレス空間は、0x00000000番地から0xffffffff番地までの全番地の集合。

アドレス変換が無ければ、
プログラムがメモリを見るときのアドレス空間「論理アドレス空間」と
ＣＰＵがメモリを見るときのアドレス空間「物理アドレス空間」とは一致する。

アドレス変換が有れば、論理アドレス空間と物理アドレス空間の対応表をＣＰＵが持つ。

セグメント方式
セグメント名は個々の変換設定の名前である。ある変換設定は論理アドレス空間と物理アドレス空間の対応関係を一つ示す。

セグメントは物理アドレス空間の区画で、ページは論理アドレス空間の区画である。

セグメンテーションは物理メモリをいくつかのセグメント（区画）に分け、１つのプログラムを１つのセグメントに格納する。
プログラムから見える０番地は、実際にはセグメントの先頭を表す。

ページングは論理アドレス空間を小さな同じ大きさのページ（区画）に分け、ページ単位で物理メモリを割り当てる。

これらセグとペジは、386以降のCPUに搭載されている。当然、これらの機能を持たないＣＰＵもある。
プログラムをメモリに配置するときに、機械語を書き換えてラベルの番地を修正する方法がある。
アドレス・リロケーション（再配置）という。

アド変できるＣＰＵでも共有ライブラリを動的に読み込んで使うにはアドレス再配置が必要になる。

### ３－２：エミュレータのorg対応

ふつうのパソコンのBIOSはプログラムを0x7c00番地に配置する。

### ３－３：プログラムの実行

この節ではメインメモリに配置されたプログラムをＣＰＵが実行する過程を説明する。
対象となるＣＰＵはx86アークテクチャのi386。
過程は大きく分けて３つ、フェッチ、デコード、実行。
フェッチ：ＣＰＵがメインメモリから命令を読み込む
デコード：命令を解釈する

表3.4　汎用レジスタの一覧

２－２：メモリ　から再掲

386には32ビット幅の汎用レジスタが8つある。
旧a,b,c,d(アキュムレジスタ、ベースレジスタ、カウントレジスタ、データレジスタ)だった、
eax、ebx、ecx、edxの４つと…（eは拡張extend）

インデックスレジスタ
esi ソースインデックス
edi ディスティネーションインデックス

特殊レジスタ
esp スタックポインタ
ebp　ベースポインタ

アキュムレータ　値を累積
ベース　メモリ番地を記憶
カウンタ　文字列の添字やループ回数を数える
データ　I/O装置の番地を記憶する

ソースインデックス　入力データの添字を記憶
ディスティネーションインデックス　　入力データの添字を記憶
スタックポインタ　スタックの先頭を指す　ほとんど特殊レジスタであり、普通の計算にはまずつかわない
ベースポインタ　スタック上の何らかのデータを指す


386のレジスタは32ビット幅だが、8ビット時代、１６ビット時代の名残りで下位１６ビットやさらに８ビットにも名前がある

eaxの下位16ビットはax、またaxの上位８ビットをah、下位８ビットをalと呼ぶ（highとlow）。
ebx、ecx、edxも同様。

もともｔと、レジスタ幅８ビットのintel8008にa、b、c、dの４レジスタあり、
intel8086で16ビットに拡張、extendでax、bx、cx、dxとなる。

esi 、edi 、esp 、ebpには8ビット区分はなく、下位１６ビットにsi、di、sp、bp、　

表3.5
eip　命令ポインタ　現在実行中の命令の番地を記憶。下位16ビットはip
eflags　フラグレジスタ　演算結果やCPU状態などを表す。下位16ビットはflags

CPUにはレジスタ以外にも演算処理を行うための加算器や論理演算器を持つ。
加算器は加算減算インクリメントデクリメントなど。条件分岐命令の直前に使うcmp命令も使う。
論理演算器はビットごとのANDやORなどの処理。
ふたつを合わせてALU算術論理演算器アリスメティックロジックユニットと言うことも
（『CPUの創りかた』を見よ）

ＣＰＵからはアドレスバスとデータバスが出てその他の装置につながる。

一般的なPC/AT互換機では、0xa0000から0xaffffまではVRAMにつながる
mov byte [0xa0000],0x0f とすれば、
アドレスバスには0x000a0000が、
データバスには0x0000000fが、
出力され、VRAMの先頭１バイトに0x0fが書き込まれる。ディスプレイには画面左上に
白点が打たれるだろう

表3.6はModR/mを詳しく見たもの。Mod　REG　R/M







### ３－４：エミュレータのMod/M対応

### ３－５：無条件分岐命令

### ３－６：call命令とスタック

### ３－７：エミュレータのcall対応

### ３－８：ローカル変数とスタック

### ３－９：フラグレジスタの条件分岐命令

### ３－１０：エミュレータの条件分岐命令対応

### ３－１１：プログラムの繰り返し

### ３－１２：デバイスアクセス

ＣＰＵの動作を理解するのは重要だが、
人間が使うシステムとしてのコンピュータを理解するためには、
それだけでは足りない。マンマシンインターフェイスがいる。

ＣＰＵと入出力装置は、Ｉ／Ｏポートを介して繋がっている。
完全なＩ／Ｏマップを得るのはいまやむつかしい。

I/Oポートのアドレスバスは、一般的なx86アークテクチャのコンピュータの場合は16ビット幅
CPUは0x0000から0xffffまでの領域を読み書きできる
mov命令ではなく、in,outというI/Oポート専用の命令を使う

in al,0x60

なら、I/Oポートの0x60番地から8ビットデータが読み込まれalレジスタに書かれる。
0x60にはKBCのデータポートが接続されているので、キーボードからの入力を1バイトのみ受け取る。

本物のI/Oの仕組みでは、inの実行時点で読み取るべきデータがないときは
空のデータか何らかのゴミデータが読みだされる。
なので、本物のI/Oをつかってデータが入力されるまで待ってから読み込むを成すには要一工夫
このまとめではその方法は略す





## chap.4 BIOSの仕組みと仕組みと実機起動

in/out命令で周辺機器を制御できるが、正しく制御するには機器固有の仕様を正しく知らねばならず難しい。
BIOSは入出力装置の差を吸収して統一したインターフェイスを提供する機能。

またエミュの実装に留まらず、作成したプログラムを実機で動かす方法も解説する。
MBRの仕組みを説明し、実際に起動イメージを作ってUSBメモリに書き込み、ＯＳに頼ることなく自作プログラムを実行する、



### ４－１：BIOS

Basic Input Output System
助けになるがすごく古いものである。
BIOSはintel8086時代の動作モード（リアルモード）でしか呼び出せない
WinもMacOSもLinuxもi286以降から導入された動作モード（プロテクトモード）で動く
またBIOSにはドライバソフトウェアを組み込んで使うような仕組みがない

いまでは無用の産物に等しいBIOSだが、重要な役割を保持している
パソコンの起動直後に機器を初期化すること
ＨＤＤから最初のプログラムローダをメモリに読み出すこと
BIOSはパソコンの起動に不可欠である

あとアセンブリ言語を学ぶときにBIOSの基礎的な入出力が役に立つ
アセンブリ言語によるプログラムを動かすにはOSのない環境が手軽
そのときにBIOSの文字表示機能がとてもありがたい

2015年、UEFIがBIOSの代わりとなる新しい規格が広まりつつある

Unified Extensible Firmware Interface
ユニファイド・エクステンシブル・ファームウェア・インタフェース

ストレージ管理には従来のMBR（Master Boot Record）に代えてGPT（GUID Partition Table）と呼ばれる仕様が導入され、
2TB（テラバイト/正確には2TiB）を超える大容量の領域を作成し、OSを導入して起動（ブート）することができる。
第二世代、第三世代のCore iシリーズの時期からマザーボードに実装されています

従来使われてきたBIOS（Basic Input/Output System）は16ビットマイクロプロセッサの時代に設計されたもので、
マルチタスク環境で利用されることを想定していない点や、メインメモリの先頭から640KB目から1MB目のわずか384KBの領域にしか配置できない点など、
現在のハードウェアやOSから見ると時代遅れで窮屈な制約が多い。
BIOSには、主にAMIとAwardという2つのプログラムに分かれます。AMIとAwardは設定画面やメニューが少し異なるだけで、実際に設定出来る項目はほとんど同じです。
AMIは、Main、Advanced、Power、Boot、Exitなどのメニュー項目があります。
Awardは、BIOS画面を開くと、メニュー画面が一覧表示されるのが特徴です。

https://www.pc-master.jp/jisaku/bios-uefi.html

これを克服するため、64ビット環境を想定して新たに設計された近代的で拡張可能なファームウェアのインターフェース仕様としてUEFIの開発が始まった。

さて、BIOSを呼び出す命令はintである。Ｃ言語のintと同じ３文字なのでまぎらわしいが。整数型はインテンジャ。こちらのintはinterrupt割り込み。
int N でN番目のサブルーチンをcallするといったもの　Nは0x00から0xffの256である


db命令
orgと同じく疑似命令である。その場所に任意のバイトを埋め込むための命令。
アセンブリ言語では表せない任意のバイト列を埋め込めるし、
埋め込んだバイト列がデータとしてのバイト列であって機械語としてのバイト列でないことが視認できる。

なお、gccは32ビットモードで動かすための機械語を作るコンパイラなので、
BIOSは16ビットなので、16ビットなBIOSを使うプログラムをＣ言語で書くのは大変
アセンブリ言語だけで書くのも手


### ４－２：BIOSの実装

リスト4.3　subroutine32.asm
はリスト4.2のプログラムを32ビット版として書き直し、
エミュ用にjmp 0を書き加えた4.3を動かせることを目標
レジスタサイズを変更した。16ビット用で使ったsiやbxをesiやebxに。
8ビットレジスタは16でも32でも使えるのでalやahそのまま

ANSIエスケープシーケンスの色番号
０黒、１赤、２緑、３黄、４青、５マゼンタ（紫がかかった濃いピンク）、６シアン（水色に近い青緑色）、７白

BIOSのひと文字表示機能ではblレジスタに文字色を指定する
指定できる色は４ビットで１６色、最上位ビットは輝度

winのMS-DOSはANSIエスケープシーケンス非対応
フリーソフトANSICONをつかえ


### ４－３：割り込み

アセンブリ言語のint(errupt)とcallは似てる面もあるが、本当はかなり異なる。
なお、ここではリアルモード（intel8086互換動作モード）での割り込みのみ扱う、
プロテクトモード（i286）は省く

callはいつ呼び出すか決まっているが、割り込みはいつ呼び出されるか不明である。
CPUはひとつ命令を実行するたび、割り込み要求が来ているかを確かめる。
来ていなければ次の命令を実行し、要求が来ていれば現在の作業を後で再開できるように
スタックにflagsの内容を保存しサブルーチンを呼び出す。

割り込みで使うサブルーチンは、いつ呼び出されるか分からないことを前提に処理を書く。
flasgs以外のレジスタはプログラマの責任でサブルーチンの呼び出し前後で保持せねばならない。

呼び出し元の戻るには、callのときはretだが、割り込みはiret（flagsの復元も行う）

ソフトウェア割り込みに対し、入出力装置からの割り込みはハードウェア割り込みという。

intがcallと違うのは、サブルーチンの名前や番地が分からなくても割り込み番号が分かればハンドラを呼び出せる

一般に、intは割り込み番号をオペランドとして取る２バイトの命令
int 0x10はcd 10
int 0xffはcd ff
割り込み番号３だけは特別で、ccという１バイト命令となる
デバッガでブレークポイントを設定するによい
（1バイト命令なので、前後の命令に迷惑を与えない　なぜか？）

### ４－４：ブートセレクタ

BIOSはふつうHDDから起動させる。対応してる機器なら何でも起動できるが、
光学ドライブから起動する仕組みは、HDDやUSBメモリから起動する仕組みに比べると特殊なので、
本書では後者のみ記す。別書（『30日OS自作』）にはフロッピーディスクから起動する手順と、
CDから起動する手順が書かれている。そちらを当たってほしい。

HDDはブロックデバイスといい、データ管理を512バイトないし4096バイトで行っている。
この１ブロックを物理セクタと呼ぶ。メモリは１バイト単位でデータの読み書きするが、
HDDはセクタ単位でのみ行う。なので、BIOSにHDDの読み書きを指示するときは、
「メモリの0x8000番地からの領域に63セクタ目を読み込め」といった指示を出す。

2009年頃までのHDDは１物理セクタ＝512バイトばかりだった、のちは4096バイトの製品も作られた。
製品ごとにセクタサイズが異なると、上のBIOSへの指示にある「63セクタ目」の内容が異なってくる。
それゆえ、HDDを使う側は物理セクタの大きさがどうあろうと512バイト固定の「論理セクタ」で場所を指示する。
これをBig Sectorという規格である。以後、この説明文で現れる「セクタ」は「論理セクタ」とする。

さて、BIOSはそのデバイスが起動可能かどうかどうやって調べるか。
優先度の高いデバイスからしらべてゆき、
最初に見つけた起動可能デバイスの先頭セクタを、
メモリの0x7c00から始まる512バイトの領域へコピーする。
BIOSの先頭セクタ読み込みプログラムは最後にメモリの0x7c00へジャンプする。

なお、そのデバイスの先頭セクタの最後の２バイト、
それは先頭を0番地とすると510番地と511番地だが、
そこに0x55,0xaaが書き込まれていたら、起動可能と判断される。

そのため、ブロックデバイスの先頭セクタは一般にブートセクタと呼ばれる。
複数パーティションをもつHDDのような機器のブートセクタをMBR（マスターブートレコード）、
フロッピーディスクやUSBメモリなの１つのパーティションのみ持つ機器のそれを
PBR（パーティションブートレコード）と呼んで区別することもある。

MBRの詳細について

先頭から446バイトは機械語を書き込む領域で、その後はパーティションテーブルが続く
テーブルは４つのパーテ…情報を持ち、各テーブル16ばいとでは以下に示す構造になっている。
起動プログラムが起動可能フラグが0x80のパーティションを探し、その内容を読み込んで起動する

+1　起動可能フラグ　可能なら0x80、不可能なら0x00
＋１～＋３　開始位置（CHS）　CHS方式で示したパーティションの開始位置
＋３　パーティションタイプOSやファイルシステムの種類
＋５～＋７　CHS方式で示したパーティションの終了位置
＋８～＋ｂ　終了位置（LBA）　LBA方式で示したパーティションの開始位置
＋Ｃ～＋ｆ　パーティションのセクタ数の合計

CHS方式とLBA方式のちがい。HDDのなかでセクタを特定するための番号を振る方法であり、
２種類あるのは単なる歴史的経緯のせい、いまでは実質的にLBAしか使われていない
ロジカルブロックアドレスといい、先頭セクタを０とした通し番号。
パーテ…テーブルは32ブットでLBAを設定できる。512バイトかける2^32≒2TBまでのセクタを指定できる。

なお、これがHHDの2TBの上限の壁であり、それ解決して2TB以上のHDDを扱えるGPT(GUID Partition Table)方式に移行しつつある。

CHS方式についても少し触れる。シリンダ(Cylinder)、ヘッド(Head)、セクタ(Sector)。MS-DOS時代のもの。
BIOSの仕様では、1024シリンダー×256ヘッド×63セクターが上限で、1セクターが512バイトであるため、約8.4Gバイト(約7.9Giバイト)が上限となる。
一方、ATA(IDE)の仕様では65,536シリンダー×16ヘッド×255セクターが上限で、1セクターが512バイトであるため約137Gバイト(127.5Giバイト)が上限となる。
しかし実際にCHSでアクセスするには全てのパラメーターが制限内に収まっていなければならないため、
実際は1024×16ヘッド×63セクターまでしか使うことができず、結果として約528Mバイト(504Miバイト)が限界となる

MBRはOSやパーティションのフォーマット形式に依存しない。
PBRはOSやパーティションのフォーマット形式に依存したローダプログラムを含む。






### ４－５：PBRを見てみよう

### ４－６：実機で動かしてみよう


## Appendix

### Ａ：開発環境のインストールと構成

### Ｂ：ASCIIコード表




