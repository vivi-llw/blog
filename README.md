# Blog of Vivi

Blog's source code.

* blog： http://vivi.github.io 
* **Use**:  hugo + travis CI + github page 

## info
blog文章位于content/portfolio中，采用markdown格式，当新增文档，上传到仓库时候，travis会自动化构建，部署到账号的github page中。

要想让travis自动推送代码到github page，需要在github  settings/Developer settings/Personal access tokens 中create一个token，并且在travis.yml中配置。
