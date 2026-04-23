# texture1: テクスチャ第５回「テクスチャ座標」サンプルプログラム

## 1. 概要

このプログラムは、OpenGL における「テクスチャマッピング (Texture Mapping)」の基礎を学ぶための、学生向けのサンプルプログラムです。本プログラムは、以下のブログ記事の解説に沿って学習を進めるための雛形として提供されています。

- [テクスチャ第５回：テクスチャ座標](https://tokoik.github.io/blog/opengl/%E3%83%86%E3%82%AF%E3%82%B9%E3%83%81%E3%83%A3/2004/09/17/texture.html)

今回は、3Dコンピュータグラフィックスにおける「テクスチャマッピング」において、**テクスチャ座標**に焦点を当てます。テクスチャ座標は、テクスチャを物体表面に貼り付ける際に、テクスチャのどの部分を物体のどの頂点に対応させるかを指定するための座標です。

## 2. ビルド方法

このプログラムは CMake を用いてビルドを構成しています。各プラットフォームごとの手順は以下の通りです。なお、プログラムをビルドするためのバイナリディレクトリは、バージョン管理ファイル（.gitignore）の設定に合わせて build という名前にします。

### Windows の場合

Visual Studio 2022 と CMake がインストールされている環境を想定しています。
コマンドプロンプトまたは PowerShell を開き、プログラムのソースコードがあるディレクトリで以下のコマンドを実行してください。

```cmd
mkdir build
cd build
cmake -G "Visual Studio 17 2022" ..
```

上記を実行すると、build ディレクトリ内に Visual Studio 用のソリューションファイル (texture1.sln) が作成されます。これを Visual Studio で開き、上部のメニューからビルドを実行してください（依存ライブラリである freeglut やヘッダファイルなどは CMake が自動的にダウンロードして組み込みます）。

### macOS の場合

Xcode および CMake がインストールされている環境を想定しています。
ターミナルを開き、プログラムのソースコードがあるディレクトリで以下のコマンドを実行してください。

```bash
mkdir build
cd build
cmake -G Xcode ..
```

上記を実行すると、build ディレクトリ内に Xcode 用のプロジェクトファイル (texture1.xcodeproj) が作成されます。これを Xcode で開いてください。

### Ubuntu Linux の場合

ビルドには、C++コンパイラ、CMake、および OpenGL と freeglut の開発用パッケージが必要です。
（例: sudo apt install build-essential cmake freeglut3-dev pkg-config）

ターミナルを開き、ソースコードがあるディレクトリで以下のコマンドを実行してください。

```bash
mkdir build
cd build
cmake ..
make
```

## 3. 使い方

### プログラムの起動方法

*   **Windows**: Visual Studio 上で「ローカル Windows デバッガー」をクリックして実行するか、またはコマンドプロンプトから以下のコマンドで起動します。

    ```cmd
    cd build\Debug
    texture1.exe
    ```

*   **macOS**: Xcode 上で左上の「Run（再生ボタン）」をクリックするのが最も簡単です。これにより texture1.app アプリケーションバンドルとして自動的に実行されます。

*   **Ubuntu Linux**: ターミナルから以下のコマンドで実行ファイル（バイナリ）を直接起動します。

    ```bash
    cd build
    ./texture1
    ```

### 操作方法

*   **マウスの左ドラッグ**: 画面上の空間をドラッグすると、表示されている箱（物体）を自由に回転させることができます。多様な角度から、テクスチャがどのように貼り付けられているかを確認してください。

*   **キーボード操作**: `Q` キー、または `ESC` キー を押すとプログラムが終了します。

プログラム起動中は、箱の表面に貼り付けられた絵柄（テクスチャ）が自動的に回転し続けるアニメーションが表示されます。これは物体そのものが回転しているのではなく、テクスチャの貼り付け方をプログラム上で変化させているためです。

## 4. 解説

プログラムのソースコード（主に main.cpp と box.cpp）をもとに、このプログラム内で行われているテクスチャマッピングのアルゴリズムと処理手順を解説します。

### A. テクスチャの読み込みと割り当て (main.cpp 内の init 関数)

1.  **ファイルの読み込み:**

    `fopen()` と `fread()` を使って、生画像データ (tire.raw) を配列 `texture` に読み込みます（256×256ピクセル、RGBA各1バイト）。

2.  **OpenGLへの転送:**

    [`glTexImage2D()`](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glTexImage2D.xhtml) 関数を使って、配列に読み込んだ画像データを OpenGL のシステム（VRAMなど）に**2Dテクスチャ**として登録します。

3.  **各種パラメータの設定:**

    [`glTexParameteri()`](https://registry.khronos.org/OpenGL-Refpages/gl4/html/glTexParameter.xhtml) 等を使用し、テクスチャを拡大縮小して貼り付ける際の補間方法（`GL_NEAREST` = ニアレストネイバー、最近傍法）、およびテクスチャ座標が定められた範囲をはみ出した際の処理（`GL_CLAMP`）を設定しています。また `GL_MODULATE` を指定することで、物体の元の色や陰影計算の結果と、テクスチャの色を掛け合わせて描画するよう指示しています。

### B. アルファテストによる透過処理 (main.cpp)

本プログラムで読み込んでいる画像には、透明度（アルファチャンネル）情報が含まれています。
`init()` 関数内で [`glAlphaFunc(GL_GREATER, 0.5)`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glAlphaFunc.xml) と設定し、描画直前に [`glEnable(GL_ALPHA_TEST)`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glEnable.xml) を有効にしています。これは「アルファ値が 0.5 より大きい（不透明に近い）ピクセルだけを描画し、透明な部分は描画しない」という処理（アルファ抜け）であり、これにより形状が複雑に切り抜かれたような表現を簡単に実現しています。

### C. テクスチャを貼る物体の描画 (box.cpp 内の box() 関数)

実際に直方体を描画している部分です。[`glBegin(GL_QUADS)`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glBegin.xml) から [`glEnd()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glEnd.xml) の間で、6つの面（四角形）を描画しています。
ここで最も重要なのは、[`glVertex3dv()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glVertex.xml)（頂点座標の指定）を呼び出す**直前**に、[`glTexCoord2dv()`](https://registry.khronos.org/OpenGL-Refpages/gl2.1/xhtml/glTexCoord.xml) を使って**テクスチャ座標**を指定している点です。

テクスチャ座標は本来、画像の左下を (0.0, 0.0)、右上を (1.0, 1.0) として表しますが、このプログラムではあえて (0.0, 0.0) から (2.0, 2.0) という領域外の値を設定しています。通常であれば画像が繰り返しマッピングされますが、`main.cpp` の初期化処理で `GL_CLAMP` を指定しているため、画像の繰り返しは行われず、領域外は最外周の色で塗りつぶされる（または透明になる）ようになっています。

### D. テクスチャのアニメーション (main.cpp 内の display() 関数)

描画するたびに呼ばれる `display()` 関数内では、アニメーションのフレーム（時間 `t`）を更新しており、その時間をもとに**テクスチャ行列 (`GL_TEXTURE`)** に対して回転変換を適用しています。

```c
glMatrixMode(GL_TEXTURE);
glLoadIdentity();
glTranslated(0.5, 0.5, 0.0);
glRotated(t * 360.0, 0.0, 0.0, 1.0);
glTranslated(-1.0, -1.0, 0.0);
```

`box.cpp` で設定したテクスチャ座標 (0.0, 0.0) ～ (2.0, 2.0) の中央は (1.0, 1.0) です。そのため、まず `glTranslated(-1.0, -1.0, 0.0)` でテクスチャ座標の中央が原点に来るように平行移動し、その原点を中心に `glRotated` で回転させます。最後に `glTranslated(0.5, 0.5, 0.0)` を行い、回転の基準とした原点がテクスチャ画像自身の中心 (0.5, 0.5) に一致するように戻しています[^1]。

[^1]: OpenGL の行列操作は、コードの記述とは**逆順**に適用されることに注意してください。

このようにテクスチャ行列を操作することで、頂点座標やテクスチャ座標の配列自体を書き換えることなく、ポリゴンの中央に配置された画像だけがくるくると回転するアニメーションを実現しています。
