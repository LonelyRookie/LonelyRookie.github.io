
## 部署

```bash
hexo clean
hexo generate
hexo deploy
```

或简写：

```bash
hexo cl && hexo g && hexo d
```

## 日常发布流程

```bash
# 1. 创建文章
hexo new "文章标题"

# 2. 编辑文章（使用 Markdown 编辑器）

# 3. 本地预览
hexo clean && hexo generate && hexo server

# 4. 确认无误后部署
hexo clean && hexo generate && hexo deploy
```

注：使用`hexo deploy`命令后，会把静态资源文件推送到`master`分支。

```yml
deploy:
  type: git
  repo: git@github.com:LonelyRookie/LonelyRookie.github.io.git
  branch: master
```

--- 

`master`分支存放的是打包后的静态资源文件

`feature/dev`分支存放的是工程代码


## 将本地工程推送到远程仓库

```shell
git push origin HEAD:feature/dev
```
