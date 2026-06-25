# Mobile Design Rebuild Skill

一个用于 Codex 的移动端设计还原 skill。适用于用户提供一张或多张移动端设计图，并要求 Codex 对 iOS、Android 或 H5 的指定页面进行高保真还原、重构或重新设计的场景。

## 目录结构

```text
mobile-design-rebuild-skill/
  README.md
  LICENSE
  mobile-design-rebuild/
    SKILL.md
    agents/
      openai.yaml
```

## 能力范围

- 支持 iOS、Android、H5 单端或多端设计还原。
- 先输出 HTML/JSON/Markdown 规格包，确认后再开发。
- 根据设计图尺寸和目标端分别计算 pt、dp/sp、CSS px/rem。
- 要求缺失图片、图标、纹理、装饰元素通过生图生成，不用 SVG、系统图标或代码绘图近似替代。
- 支持主线程统筹、子线程执行生图/裁剪/局部实现的工作流。
- 按目标端使用 Simulator、Emulator 或 Browser/Playwright 做视觉验证。

## 安装

从 GitHub 安装时，把 `OWNER/REPO` 替换成实际仓库：

```bash
python3 ~/.codex/skills/.system/skill-installer/scripts/install-skill-from-github.py \
  --repo OWNER/REPO \
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
