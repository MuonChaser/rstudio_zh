# locdiff

locdiff 是一个命令行工具，用于生成 CSV 文件，显示在 RStudio IDE 仓库的两个提交哈希之间新增或修改的所有英文 UI 字符串。

该工具比较 RStudio 仓库的两个克隆实例，一个名为 "old"，一个名为 "new"。你需要自行创建这两个实例，并在每个实例中检出所需的提交哈希。

它还会显示每个字符串最近的法语翻译（如果有），以便更容易识别需要处理的内容。

## 用法

该工具需要 `node.js` 环境，支持 Mac、Windows 或 Linux。推荐使用 node.js 22.13.1 或更高版本，但任何较新的 node.js 都可以运行。

```bash
cd rstudio/src/gwt/tools/locdiff
git clone git@github.com:rstudio/rstudio
mv rstudio old
git clone git@github.com:rstudio/rstudio
mv rstudio new
cd old
git checkout 9f796939
cd ../new
git checkout main
cd ..
npm i
npm start
```

结果会写入 `locdiff.csv`，可在 Excel 等工具中打开。
要在 Excel 中正确显示法语字符串，需要导入（而不是直接打开）该 CSV 文件，并指定为逗号分隔且采用 Unicode / UTF-8 编码，否则法语字符串可能会乱码。

如果你有 Apple Numbers，可以直接加载该文件。

### 参数

--only-changed - 只输出已更改或新增的字符串

### 其他检查

请检查所有法语属性文件，确保没有未转义的单引号字符。可在所有 *._fr.properties 文件中搜索正则表达式 (?<!')'(?!')。
