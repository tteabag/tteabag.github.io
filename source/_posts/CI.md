---
title: CI
date: 2024-04-28 13:57:05
tags:
---

# UT涉及到内容

- Style Check（样式）

- Cyclomatic（圈复杂度分析）

- Duplicatie（重复代码检测）

- Code Coverage（代码覆盖率）

- 代码统计

- 单元测试报告

<!-- more -->

### Style Check

Style Check有OCLint、SwiftLint两种语言的检测

###### OCLint [官网](https://oclint.org)

步骤：

1. 生成 xcodebuild.log
   
   `xcodebuild`

2. 生成 compile_commands.json
   
   `oclint-xcodebuild <xcodebuild.log>`
   
   或者
   
   `xcpretty -r json-compilation-database -o compile_commands.json`

3. 生成结果 oclint.xml
   
   `oclint-json-compilation-database [-e exclude/file/path] -- -report-type pmd -o save/file/path [-disable-reult rule] [-rc rule=value]`

###### SwiftLint

`swiftlint | tee path/swiftlint.xml`



### Cyclomatic

圈复杂度(Cyclomatic Complexity Number, CCN)分析主要使用三方库 [lizard](https://github.com/terryyin/lizard) 生成结果包含CCN、Ncss(Non Commenting Source Statements) 指标

`lizard path/to/dict --xml  > result/path/cyclomatic_complexity.xml`

### 

### Duplicatie

重复代码检测使用三方库 [PMD](https://pmd.github.io)

```shell
cd $HOME
wget https://github.com/pmd/pmd/releases/download/pmd_releases%2F7.1.0/pmd-dist-7.1.0-bin.zip
unzip pmd-dist-7.1.0-bin.zip
alias pmd="$HOME/pmd-bin-7.1.0/bin/pmd"
pmd check -d /usr/src -R rulesets/java/quickstart.xml -f text
```



### Code Coverage

代码覆盖率有两种方式：

1. LLVM

2. [SLATHER](https://github.com/SlatherOrg/slather)

###### LLVM

LLVM方式依赖单元测试，步骤如下：

1. 执行单元测试生成必要的文件
   
   ```shell
   xcodebuild test \
   -workspace WorksOne.xcworkspace \
   -scheme ${xcode_build_scheme} \
   -configuration Debug \
   -enableCodeCoverage YES \
   CODE_SIGN_IDENTITY="" CODE_SIGNING_REQUIRED=NO \
   GCC_GENERATE_TEST_COVERAGE_FILES=YES \
   SWIFT_OPTIMIZATION_LEVEL="-Osize" \
   OTHER_SWIFT_FLAGS="-DTARGET_WorksOne -DTARGET_TEST" \
   ARCHS="x86_64" \
   2>&1 | ocunit2junit
   ```

    2. 

```shell
xcrun llvm-cov report 
-instr-profile path/to/Coverage.profdata
-object path/to/excutefile > path/to/result
```

tip: Coverage.profdata文件在`Build` 目录下

###### SLATHER

```shell
slather coverage
--build-directory path/to/Build/Products
--binary-file path/to/binary
--cobertura-xml
--output-directory result/path/dir
--scheme OCLintDemoTests
path/to/*.xcodeproj
```

可以使用 --ignore 指定忽略文件



### 代码统计

代码统计有两个工具：

1.[sloccount](https://dwheeler.com/sloccount/)

`sloccount --duplicates --wide --details OneApp > target/sloccount-reports/sloccount.sc`

2.[cloc](https://cloc.sourceforge.net)

`cloc --by-file --xml --out=target/sloccount-reports/cloc.xml --xml --exclude-ext=xml,perl,json,html,xslt OneApp`
