# setup-nebula

配置 `@zeniein/*` 私仓认证,并可选执行 `nebula-pack init` 生成 `nebula.config.ts`。

`run-init: 'true'`(默认)首次接入生成 config;后续 CI 改 `run-init: 'false'` 只复用认证。详细 inputs / outputs 见 `action.yml`。

## 用法

`outputs.init-outcome` 在 `run-init: 'false'` 时值为 `skipped`,可用于条件分支。

```yaml
- uses: actions/checkout@v4
- uses: have1dot6/nebula-actions/setup-nebula@<ref>
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
- run: npx @zeniein/nebula-pack-cli build --platform android
```

`<ref>` 建议固定为具体 tag(如 `@v1.0.0`)或 SHA;调试时也可使用相对路径 `- uses: ./setup-nebula`。

## 备注

- `github-token` 为必需 input,需由调用方传入(通常为 `${{ secrets.GITHUB_TOKEN }}`);`registry-token` 默认回退到 `github-token`,本地可自填 PAT。
- `init` 是幂等的:config 已存在时仅 warn,不报错。
