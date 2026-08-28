# setup-android

构建并打包 `mobile` 项目到 Android(默认 release;`debug: 'true'` 时打 debug 包),并把产物作为资源上传到一个 **预先存在的** GitHub Release。

JDK 与 Android SDK 由本 action 安装,默认对齐项目模板:`compileSdk=35`、`buildTools=35.0.0`、JDK 17;随后依次执行:

1. `nebula-pack build -p mobile`
2. `nebula-pack package -p mobile -t android [-d]`
3. 在 `${outputDir}/mobile/android` 下按 `artifact-pattern` 找到 APK/AAR,作为独立 asset 一起 `gh release upload` 到目标 release

注意:`${outputDir}/mobile/android` 是 nebula-pack package 出来的 **原生 Android 工程目录**(含 `settings.gradle`、`gradlew`、`app/`、`*/build/outputs/` 等),不是构建产物目录本身;真正的 APK/AAR 在它里面的 `*/build/outputs/` 子树下,需要 glob 找到。

Gradle 走工程内 wrapper(`gradlew`),本 action 不下载 Gradle。详细 inputs 见 `action.yml`。

## 前置

- 同一 job 内必须先调用过 `setup-nebula`(提供 Node、写入私仓认证、全局安装 `nebula-pack` CLI,首次跑一次 `nebula-pack init` 生成 `nebula.config.ts`)
- 目标 Release 必须已存在(由 release-please / 手动 `gh release create` / 其它流程预先创建);本 action 不会自动创建 Release
- runner 需自带 `gh` CLI(GitHub-hosted runner 默认已装;自建 runner 需自己确保)

## 用法

典型编排:打 release APK 并上传到当前 repo 的 `v1.0.0` release:

```yaml
- uses: have1dot6/nebula-actions/setup-nebula@<ref>
- uses: have1dot6/nebula-actions/setup-android@<ref>
  with:
    release-tag: v1.0.0
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

打 debug 包(只发 debug 包的 APK,过滤掉其它):

```yaml
- uses: have1dot6/nebula-actions/setup-nebula@<ref>
- uses: have1dot6/nebula-actions/setup-android@<ref>
  with:
    release-tag: v1.0.0-debug
    debug: 'true'
    artifact-pattern: '**/build/outputs/apk/debug/*.apk'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

`nebula.config.ts` 不在仓库根(`outputDir: './dist'` 解析自 `./mobile/nebula.config.ts`):

```yaml
- uses: have1dot6/nebula-actions/setup-nebula@<ref>
- uses: have1dot6/nebula-actions/setup-android@<ref>
  with:
    release-tag: v1.0.0
    working-directory: ./mobile
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

同时上传 APK + mapping.txt(用于 crash 反混淆):

```yaml
- uses: have1dot6/nebula-actions/setup-android@<ref>
  with:
    release-tag: v1.0.0
    artifact-pattern: '**/*.apk **/mapping.txt'
    github-token: ${{ secrets.GITHUB_TOKEN }}
```

跳过上传(只想 build/package):

```yaml
- uses: have1dot6/nebula-actions/setup-nebula@<ref>
- uses: have1dot6/nebula-actions/setup-android@<ref>
  with:
    release-tag: v1.0.0
    upload: 'false'
```

跨 repo 上传(产物推到 `have1dot6/dist-packages` 而非 action 执行 repo),需自备 PAT:

```yaml
- uses: have1dot6/nebula-actions/setup-nebula@<ref>
- uses: have1dot6/nebula-actions/setup-android@<ref>
  with:
    release-tag: v1.0.0
    release-repo: have1dot6/dist-packages
    github-token: ${{ secrets.CROSS_REPO_PAT }}
```

## 备注

- `artifact-pattern` 相对于 `${outputDir}/mobile/android`(解析自 `working-directory`),支持 `**` 递归;多个 pattern 用空格分隔。默认值 `**/*.apk **/*.aar` 会匹配整棵 android 子树里的所有 APK 和 AAR。
- 匹配不到任何文件时直接 `::error::` 失败,避免空上传被静默吞掉。
- `release-repo` 默认 `${{ github.repository }}`(即 action 执行 repo);`upload: 'true'` 时必须显式传 `github-token`,同 repo 传 `${{ secrets.GITHUB_TOKEN }}`,跨 repo 必须用 PAT。
- `gh release upload --clobber` 会覆盖同名 asset,适合 release 重跑场景。
- 匹配到的所有文件会以原文件名挂到 release;若想重命名,请在外层 workflow 提前处理。
