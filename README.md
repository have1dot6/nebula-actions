# nebula-actions



## 包含的 Action

| Action | 作用 | 适用 runner |
|---|---|---|
| [`setup-node`](./setup-node) | Node.js + pnpm + 托管缓存 | 任意 |
| [`setup-android`](./setup-android) | JDK + Android SDK + `nebula-pack build/package`,出 Android 包并 `gh release upload` 到指定 release(`debug: 'true'` 打 debug) | Linux / macOS / Windows |
| [`setup-nebula`](./setup-nebula) | 私仓认证(`@zeniein/*`)+ `nebula-pack init` 脚手架 | 任意(Node 可用即可) |

各 action 的输入参数与示例见各自目录下的 `README.md`。

## 引用方式

### 跨仓库引用(推荐)

在其它仓库的工作流里,通过 GitHub 直接引用本仓库的子目录:

```yaml
- uses: have1dot6/nebula-actions/setup-node@<ref>
```

`<ref>` 建议固定为具体 tag(如 `@v1.0.0`)或 SHA;跟踪 main 分支可以写 `@main`,但会随仓库演进而漂移。

### 本仓库内自引用

调试或本仓库内复用时,可使用相对路径引用子目录:

```yaml
- uses: ./setup-node
```

## 编排示例

一次完整打包(单平台,release + 上传到当前 repo 的 `v1.0.0` release):

```yaml
jobs:
  build-android:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: have1dot6/nebula-actions/setup-nebula@<ref>
      - uses: have1dot6/nebula-actions/setup-android@<ref>
        with:
          release-tag: v1.0.0
```

`setup-nebula` 负责前置的 Node、私仓认证与全局 `@zeniein/nebula-pack-cli`(默认还会跑一次 `nebula-pack init` 生成 `nebula.config.ts`);`setup-android` 复用这些前置准备,完成 Android 链路:JDK + Android SDK 安装 → `nebula-pack build -p mobile` → `nebula-pack package -p mobile -t android` → 在 `${outputDir}/mobile/android`(默认 `./dist/mobile/android`,这是 nebula-pack package 出来的原生 Android 工程目录)下按 `artifact-pattern` glob(默认 `**/*.apk **/*.aar`)找 APK/AAR,`gh release upload <tag> <files...>` 到目标 release。

打 debug 包:

```yaml
- uses: have1dot6/nebula-actions/setup-android@<ref>
  with:
    release-tag: v1.0.0-debug
    debug: 'true'
```
