# Mobile Design Rebuild Skill

语言 / Language: [中文](./README.zh-CN.md) | [English](./README.en.md)

`mobile-design-rebuild` 是一个用于 Codex 的移动端设计还原 skill。适用于用户提供一张或多张移动端设计图，并要求 Codex 对 iOS、Android 或 H5 的指定页面进行高保真还原、重构或重新设计的场景。

这个 skill 的核心思路是：先把设计图转成可确认的 HTML/JSON/Markdown 规格包，用户确认后再进入素材生成、代码实现和多端视觉验证。

## 能力范围

- 支持 iOS、Android、H5 单端或多端设计还原。
- 先输出 HTML/JSON/Markdown 规格包，确认后再开发。
- 根据设计图尺寸和目标端分别计算 iOS pt、Android dp/sp、H5 CSS px/rem。
- 缺失图片、图标、纹理、装饰元素必须通过生图生成，不使用 SVG、系统图标或代码绘图近似替代。
- 子线程生成的图片必须保存到主线程可读取的共享目录，并回传候选图路径、`handoff.json` 和审阅预览。
- 支持主线程统筹、子线程执行生图/裁剪/局部实现的工作流。
- 按目标端使用 iOS Simulator、Android Emulator 或 Browser/Playwright 做视觉验证。

## 目录结构

```text
mobile-design-rebuild-skill/
  README.md
  README.zh-CN.md
  README.en.md
  LICENSE
  mobile-design-rebuild/
    SKILL.md
    agents/
      openai.yaml
```

## 安装

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo 799536960/mobile-design-rebuild-skill \
  --path mobile-design-rebuild
```

安装后重启 Codex，让新 skill 生效。

## 使用

在 Codex 中调用：

```text
Use $mobile-design-rebuild 处理这些移动端设计图和目标页面：先判断目标端并创建 HTML/JSON/MD 规格包给我确认；确认前不要生图或改代码。
```

## 本地校验

```bash
python3 ~/.codex/skills/.system/skill-creator/scripts/quick_validate.py ./mobile-design-rebuild
```

## 发布前检查

- 确认 `mobile-design-rebuild/SKILL.md` 中没有本机路径、客户信息、私有项目名或密钥。
- 确认 `mobile-design-rebuild/agents/openai.yaml` 的默认提示词和技能正文一致。
- 确认 `quick_validate.py` 校验通过。
