### Overview

`radium-docs` [文档网站](http://formidable.com/open-source/radium/) 的源文件存放在 `/docs` 中。 `docs/**/docs.js` 组件被导入到 `radium-docs` 并部署。

### 部署

提交一个 pull 请求到 `master`。一旦合并，你就需要从 `radium-docs` 中运行 `update-project`，并合并到 `master` 中。在 `radium-docs` 中进行新的 到 `master` 的 push 将会触发一个部署。
