# C/C++ test設計

## 概要

Parasoft C/C++testを利用して、MISRA、AUTOSAR、CERTに準拠したソース品質を確保する。


## CMake調査


## 使い方
使うために会社のメールアドレスを登録する必要がある。

- Request [a free trial](https://www.parasoft.com/products/parasoft-c-ctest/try/) to receive access to Parasoft C/C++test's features and capabilities.

### 前提条件
- Parasoft licenseが必要
- GitHub-hosted runnerよりもself-hosted runnerの利用を推奨
  - AWSアカウントを作成

[AWS無料枠](https://aws.amazon.com/jp/free/?all-free-tier.sort-by=item.additionalFields.SortRank&all-free-tier.sort-order=asc&awsf.Free%20Tier%20Types=*all&awsf.Free%20Tier%20Categories=*all)について

オファーの種類は3種類
- 常に無料
- 12ヶ月無料
- トライアル

| サービス | 無料枠の詳細 |
| Amazon EC2 | 12か月間無料 |
| Amazon S3 | 12か月間無料 |
| Amazon RDS | 12か月間無料 |

## GitHub Actionsへの実装
### Installation
A GitHub Action for running Parasoft C/C++test to ensure code quality and compliance with MISRA, AUTOSAR, CERT, and more

- INSTALLATION
    - Copy and paste the following snippet into your .yml file.

```yml
- name: Run Parasoft C/C++test
  uses: parasoft/run-cpptest-action@1.0.1
```

### Adding the Rub C/C++test Actions to a GitHub Workflow
```yml
# Runs code analysis with C/C++test.
- name: Run C/C++test
  uses: parasoft/run-cpptest-action@1.0.1
  with:
    input: build/compile_commands.json
    testConfig: 'builtin://MISRA C 2012'
    compilerConfig: 'clang_10_0'
```

### Uploading Analysis Results to GitHub
レポート出力形式は以下3つ。
- SARIF
- XML
- HTML

静的解析結果の出力はSARIF形式が良さそう。
- [SARIFサポートについて - GitHub Docs](https://docs.github.com/ja/code-security/secure-coding/integrating-with-code-scanning/sarif-support-for-code-scanning#about-sarif-support)

[SARIF形式レポート出力](https://github.com/marketplace/actions/run-parasoft-c-c-test#generating-sarif-reports-with-cctest-20202-or-earlier)

```yml
- name: Run C/C++test
  uses: parasoft/run-cpptest-action@1.0.1
  with:
    reportFormat: xml,html,custom
    additionalParams: '-property report.custom.extension=sarif -property report.custom.xsl.file=${PARASOFT_SARIF_XSL}'
```

## 参考
- [Run Parasoft C/C++test](https://github.com/marketplace/actions/run-parasoft-c-c-test)

