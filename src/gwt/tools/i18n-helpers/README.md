# 摘要

本 README 介绍了国际化（i18n）开发流程及可用的辅助工具。

每当对 Commands.cmd.xml 进行任何更改（即使只是空白字符），都必须运行 `ant`、`ant draft` 或 `ant generate-i18n` 之一，并将所有修改过的文件与 Commands.cmd.xml 一起提交。否则会导致 RStudio 构建失败。

此流程要求已启用 Python3，并且已安装 commands.cmd.xml/requirements.txt 中的依赖（即通过 `pip install -r requirements.txt`）。

# RStudio 的 i18n

## 实现细节

i18n 通过 GWT 的 [i18n](http://www.gwtproject.org/doc/latest/DevGuideI18n.html) 功能实现，通常采用静态字符串国际化。在 RStudio 完全启用 i18n 之前，非英文语言环境仅在 `SuperDevMode` 下运行 `ant` 时启用。

要对部分代码库进行本地化，需定义扩展自 `com.google.gwt.i18n.client.Constants` 和 `.Messages` 的接口。在接口中为所有本地化文本包含带有 `@DefaultStringValue` 的 `String`，并在代码中引用这些 `String` 对象以获取文本值。要为应用添加语言环境，需添加名为 `INTERFACENAME_LOCALE.properties` 的属性文件。实现示例见 `src/org/rstudio/studio/client/application/ui/AboutDialog.java`，引用了：
* `AboutDialogConstants.java`（常量接口）
* `AboutDialogConstants_en.properties`（英文常量属性文件）
* `AboutDialogConstants.java`（消息接口）
* `AboutDialogMessages_en.properties`（英文消息属性文件）

调试时，可通过在 RStudio URL 后添加 `?locale=yourLocale`（如 `http://localhost:8787/?locale=yourLocale`）访问非英文语言环境。

GWT 提供本地化内容的顺序如下：
* 优先匹配所选语言环境（如 `?locale=en`，则优先提供 `*_en.properties`，如果可用）
* 默认语言环境（在 `RStudio.gwt.xml` 或其他 XML 文件中定义），如果可用
* `@DefaultStringValue` 的文本

GWT 建议始终包含 `@DefaultStringValue` 和至少一个 `.properties` 文件。

## 开发流程

在实现 i18n、将硬编码英文文本替换为英文属性文件中的文本时，可能难以判断哪些内容已翻译、哪些未翻译，以及翻译是否生效。为解决此问题，可使用“dev”语言环境，在开发时对文本进行明显的可视化更改。该“dev”语言环境：
* 是当前英文 `.properties` 文件的副本，所有常量和消息文本前加“@”，以便在 UI 中清晰可见
* 仅在 SuperDevMode（服务器或桌面）下启用，可通过 `http://localhost:8787/?locale=dev` 访问，生产环境不可用（见 `RStudioSuperDevMode.gwt.xml`/`RStudioDesktopSuperDevMode.gwt.xml`）
* 仅在开发时需要时生成，并随代码库提交（文件已在 `.gitignore` 中忽略）

例如，以下为“dev”语言环境示例，菜单和命令已支持 i18n，其他文本未支持：

![部分翻译的 dev 语言环境示例](./rstudio-dev-locale-example.png)

# 工具

以下工具可辅助 i18n 开发：

## create_dev_locale.sh

用于调试 i18n，通过从现有 `*_en.properties` 文件创建 `*_dev.properties` 文件，帮助直观确认哪些内容已启用 i18n。该脚本会复制英文属性文件，并在文本前加“@”。

请在 `/src/gwt/src` 目录下运行，语法为 `./create_dev_locale.sh`

## commands_xml_to_i18n.py

### 简介

根据 `Commands.cmd.xml` 中定义的所有命令和菜单，自动生成 Java 接口和属性文件，使用该文件中的英文文本。每次编辑 `Commands.cmd.xml` 后必须运行此脚本，且在检测到该文件变更并触发 `build`、`desktop` 或 `devmode` ant 目标时会自动执行。

以下用法展示了如何手动触发脚本（而不是通过 ant 构建目标），以创建英文和“dev”语言环境，如上文 `create_dev_locale.sh` 所述。

TODO：Commands.cmd.xml 中的快捷键目前无法本地化。

### 用法

详细选项请参见 `commands_xml_to_i18n.py -h`。

典型用法（在 `commands.cmd.xml` 子文件夹下）如下：

```shell
CMD_DIR="../../../src/org/rstudio/studio/client/workbench/commands/"

# 命令
# 接口（文本不加前缀）
python commands_xml_to_i18n.py "${CMD_DIR}/Commands.cmd.xml" cmd constant "${CMD_DIR}/CmdConstants.java" --package "package org.rstudio.studio.client.workbench.commands;"
# 英文（en）属性文件（文本不加前缀）
python commands_xml_to_i18n.py "${CMD_DIR}/Commands.cmd.xml" cmd properties "${CMD_DIR}/CmdConstants_ch.properties"
# （可选）开发（dev）属性文件（文本前加“@”，用于开发，见简介）
python commands_xml_to_i18n.py "${CMD_DIR}/Commands.cmd.xml" cmd properties "${CMD_DIR}/CmdConstants_dev.properties" --prefix "@"

# 菜单
# 接口（文本不加前缀）
python commands_xml_to_i18n.py "${CMD_DIR}/Commands.cmd.xml" menu constant "${CMD_DIR}/MenuConstants.java" --package "package org.rstudio.studio.client.workbench.commands;"
# 英文（en）属性文件（文本不加前缀）
python commands_xml_to_i18n.py "${CMD_DIR}/Commands.cmd.xml" menu properties "${CMD_DIR}/MenuConstants_ch.properties"
# （可选）开发（dev）属性文件（文本前加“@”，用于开发，见简介）
python commands_xml_to_i18n.py "${CMD_DIR}/Commands.cmd.xml" menu properties "${CMD_DIR}/MenuConstants_dev.properties" --prefix "@"
```

### 测试

该工具提供了一个最小测试套件，可通过 `python -m pytest ./test_command.py` 运行。

