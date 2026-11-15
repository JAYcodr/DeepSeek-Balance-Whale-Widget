# 小鲸鱼 · OpenCode Go 额度挂件

DSH（DeepSeek Harness）网页右下角的小鲸鱼：**装好就能看到你 OpenCode Go 订阅的额度**，点一下鲸鱼就能看 5 小时 / 周 / 月 三个窗口用了多少、什么时候重置。

这个仓库是 [MeteorNOX/DeepSeek-Balance-Whale-Widget](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget) 的 fork：界面、动效、泡泡系统全部保留，只把**默认额度来源换成 OpenCode Go**，并把第一次使用的配置压到一张卡片。

> 原版面向 DeepSeek 官方余额（余额 / 今日已用 / 峰谷定价）；这个分支面向 OpenCode Go 订阅 —— 没有余额，只有窗口额度，所以默认不查 DeepSeek、不发探针、不用先建模型。

## 装

```sh
dsh plugin --profile web add github:JAYcodr/DeepSeek-Balance-Whale-Widget
```

桌面端（Electron 客户端）在会话里让 DSH 自己装，或按 DSH 文档的「本地目录安装」走。

装完刷新网页，右下角出现小鲸鱼。

## 第一次用（30 秒）

1. 打开 [opencode.ai](https://opencode.ai) 控制台 → Go 套餐 → 复制 API key（`sk-` 开头）。
2. 刷新 DSH 网页：没有 key 时小鲸鱼会自动弹一张极简卡片 → 把 key 粘进去 → 「保存并读取额度」。
3. 点一下鲸鱼，气泡里就是：

   ```text
   OpenCode Go 额度
   5h 1% · 周 37% · 月 28%
   重置 3h12m · 2d3h · 12d
   ```

要换 key：右键鲸鱼 → 菜单 → `OpenCode Go → 设置`。

key 存在 DSH 凭据里（名字 `OPENCODE_GO_API_KEY`），插件目录不落明文；环境变量里已经有同名 key 也能直接用，和别的插件共享同一份。

## 它做什么 / 不做什么

- **只读订阅额度**：`https://opencode.ai/zen/go/v1/usage`（5 小时 / 周 / 月 三个窗口的已用百分比 + 重置时间）。
- **不发探针**：不会为了测线路真的去请求一次模型，额度刷新只打额度接口本身。
- **接口失败只显示 `--`**：不弹错误、不影响鲸鱼本体。
- **不碰 DeepSeek 官方余额**：那套功能还在（峰谷、充值校正、DeepSeek 模型），只是默认不启用；想用就在菜单里自己加 DeepSeek 模型。
- **零额外配置**：全新安装时宿主会自动种一条 `OpenCode Go` 模型，不需要去「新增模型」里挑厂商模板。

## 别的都在

除了默认值和首屏，上游的能力没有删：

- 点击序列 / 泡泡模块（文本、随机语句、图片、金额、额度…）、逐模块字体与配色、拖拽排版；
- 拖拽吸附、翻转、Q 弹玩偶、按压音效与每轮消耗提示；
- 资源管理、用量记录、余额预警（对 OpenCode Go 不适用，可忽略）。

想改泡泡内容：右键鲸鱼 → 菜单 → 「自定义泡泡」；想恢复默认 → 同窗口里的「恢复默认」，会回到上面那张 OpenCode Go 额度卡。

## 数据与安全

- 全部 **26 个** `/dsh-whale/*` 路由都接入 DSH 浏览器信任栅栏（`connection.requestRejection`），默认只接受回环地址（`127.0.0.1` / `localhost` / `[::1]`），拒绝跨站请求与 Origin/Host 不同源的请求。
- 放在反向代理或局域网后面时，用环境变量声明允许的 Host：`DSHW_TRUSTED_HOSTS=host1:port,host2`（逗号分隔）。
- 仪表只把额度百分比与重置时间发给浏览器；**key 不下发、不写日志、不落插件目录**。

## 和上游的关系

- fork 自 [MeteorNOX/DeepSeek-Balance-Whale-Widget](https://github.com/MeteorNOX/DeepSeek-Balance-Whale-Widget)（MIT），原始设计与绝大部分代码归原作者。
- 本分支的差异：
  - 默认模型 = OpenCode Go 额度（宿主首次运行自动种入，见 `lib/index.js` 的 `ensureGoModel`）；
  - 首次点击的气泡 = 额度三窗口（`bubbleDefaultFirstModules`）；
  - 新增极简设置卡 + 菜单里的 `OpenCode Go → 设置` 行（`openGoSetup` / `goSetupMaybe`）；
  - 「检查更新」改读本仓库的 GitHub Releases（上游那套 npm 源会把上游版本号当成新版本推给用户）；
  - 包名 `dsh-whale-widget-go`，不发 npm，只从 GitHub 安装。
- 上游的界面与泡泡系统改进可以继续合并；涉及默认模型、默认气泡、首屏的改动会和本分支冲突，合并时需要手工取舍。

## 开发

零依赖、零构建：宿主 `lib/index.js`（ESM），浏览器半边 `assets/whale-widget.js`（普通脚本，随包加载）。

```sh
node --check lib/index.js
node --check assets/whale-widget.js
node tools/z-layer-audit.mjs assets/whale-widget.js
node tools/ci-audit.mjs --no-pack
```

## 许可

MIT，见 [LICENSE](LICENSE)；原始版权归 MeteorNOX。仓库内 `PROVENANCE.md` 记录了上游来源。
