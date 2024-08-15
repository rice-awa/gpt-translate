# 🌎 Markdown 翻訳ロボット
[![Maintainability](https://api.codeclimate.com/v1/badges/a13ea4f37913ba6ba570/maintainability)](https://codeclimate.com/github/3ru/gpt-translate/maintainability)
[![GPT Translate](https://github.com/3ru/gpt-translate/actions/workflows/gpt-translate.yml/badge.svg)](https://github.com/3ru/gpt-translate/actions/workflows/gpt-translate.yml)

[![OpenAI](https://img.shields.io/badge/-OpenAI-white?style=flat-square&logo=openai&logoColor=black)](https://openai.com/)
[![Azure](https://img.shields.io/badge/-Microsoft%20Azure-white?style=flat-square&logo=microsoftazure&color=0078D4)](https://azure.microsoft.com/en-us/products/ai-services/openai-service)
[![Anthropic](https://img.shields.io/badge/-Anthropic-black?style=flat-square&logo=anthropic&logoColor=black&color=d4a27f)](https://www.anthropic.com/)
[![Perplexity](https://img.shields.io/badge/-Perplexity-black?style=flat-square&logo=perplexity&color=black)](https://docs.perplexity.ai/)
[![Google](https://img.shields.io/badge/-Google%20gemini-white?style=flat-square&logo=googlegemini&color=white)](https://ai.google/discover/generativeai/)
[![Groq](https://img.shields.io/badge/-Groq-black?style=flat-square&logoColor=black&color=F55036)](https://groq.com/)
[![Fireworks](https://img.shields.io/badge/-Fireworks%20AI-black?style=flat-square&color=631fee)](https://fireworks.ai/)
[![Mistral](https://img.shields.io/badge/-Mistral%20AI-black?style=flat-square&color=ff7000)](https://mistral.ai/)
[![Cohere](https://img.shields.io/badge/-Cohere-black?style=flat-square&color=39594c)](https://cohere.com/)

[English](/README.md) |
[简体中文](/README/README.zh-CN.md) |
[繁體中文](/README/README.zh-TW.md) |
[Español](/README/README.es.md) |
[हिंदी, हिन्दी](/README/README.hi.md) |
[한국어](/README/README.ko.md) |
[日本語](/README/README.ja.md)

この GitHub アクションは、複数の AI モデルを使用して、あなたの markdown ファイルを多言語に翻訳します。

> [!Important]
> 新機能：**複数のプロバイダーからの AI モデル✨**  \
> OpenAI に限定されず、複数の AI モデルプロバイダーをサポートするようになりました。  \
> サポートされているプロバイダーの完全なリストと詳細については、[リリースノート](https://github.com/3ru/gpt-translate/releases/tag/v1.2.0-beta)をご覧ください。

<br/>

<details><summary>🧐 現在の状況</summary>
<p>

- このアクションは、**markdown(`.md`)、markdown-jsx(`.mdx`)、json(`.json`) ファイル**の翻訳のみをサポートしています。

- このコマンドは、**リポジトリの書き込み権限**を持つ個人のみが実行できます。

これらの制限により、信頼できない第三者による API の悪用を防ぎます。

</p>
</details> 

## 🔧 設定

### リポジトリ設定

#### 1. 設定 > Actions > 一般

- `読写権限`を有効にする
- `GitHub Actions がプルリクエストを作成および承認できるようにする`を有効にする
  ![permissions](https://user-images.githubusercontent.com/69892552/228692074-d8d009a8-9272-4023-97b1-3cbc637d5d84.jpg)

#### 2. 設定 > Secrets と変数 > Actions

- [あなたの API キー](https://platform.openai.com/account/api-keys)(`OPENAI_API_KEY`)をシークレットとして設定する
  ![secrets](https://user-images.githubusercontent.com/69892552/228692421-22d7db33-4e32-4f28-b166-45b4d3ce2b11.jpg)

### GitHub Actions ワークフロー設定

#### 必須
- apiKey として OPENAI_API_KEY を提供する。
- コメントが作成されたときにトリガーするように `on` を設定する（`types: [ created ]`）。
- 事前にチェックアウトする（`actions/checkout@v3`）。

#### 推奨（不要な実行時間を最小限に抑えるため）
- コメントに `/gpt-translate` または `/gt` が含まれている場合にのみ実行するように設定する。

👇 以下は最小限のワークフローの例です：
```yaml
# .github/workflows/gpt-translate.yml
name: GPT Translate

on:
  issue_comment:
    types: [ created ]

jobs:
  gpt_translate:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: GPT Translate を実行
        if: |
          contains(github.event.comment.body, '/gpt-translate') || 
          contains(github.event.comment.body, '/gt')
        uses: 3ru/gpt-translate@master
        with:
          apikey: ${{ secrets.OPENAI_API_KEY }}
```

## 💡 使用方法

```
/gpt-translate [入力ファイルパス] [出力ファイルパス] [ターゲット言語] 
```
/gpt-translate の省略形として /gt を使用できます。

1. イシューまたはプルリクエストに `/gpt-translate` または `/gt` を含むコメントを作成します。

2.【イシューの場合】翻訳されたファイルは **プルリクエスト** として作成されます。

2.【プルリクエストの場合】翻訳されたファイルは **新しいコミットとしてプルリクエストに追加されます**。

言い換えれば、イシューにコメントを続けると、新しい PR が続けて作成されます。
PR にコメントを続けると、その PR に新しいコミットが続けて追加されます。

## 📝 例
```
/gpt-translate README.md zh-TW/README.md traditional-chinese
```
`README.md` を繁体字中国語に翻訳し、`zh-TW` ディレクトリに配置します。

### 複数ファイルのサポート

入力ファイルパスにワイルドカードを指定することで、一度に複数のファイルを翻訳できます。

以下はその例です：
```
/gpt-translate *.md *.ja.md Japanese
```
ルートディレクトリに `A.md` と `B.md` がある場合、出力は `A.ja.md` と `B.ja.md` になります。ファイル名は入力ファイルから継承されます。
任意のファイル名で出力することも検討していますが、賢いアイデアがあれば、イシューで提案してください！

詳細については、[ウェブサイト](https://g-t.vercel.app/docs/references/path-builder)をご覧ください。

## 🌐 サポートされている言語
GPT-4 または GPT-3.5 が解釈する**すべての**言語

## 🏘️ コミュニティ
- [ディスカッション](https://github.com/3ru/gpt-translate/discussions)
  - 質問があれば、GitHub ディスカッションでお気軽に質問してください :)
- [イシュー](https://github.com/3ru/gpt-translate/issues)
  - バグや新機能の提案は、GitHub イシューに提出してください

## 📃 ライセンス
MIT ライセンス