> 初始化站点：

```
fuxiaosongs-Air:dev fuxiaosong$ hugo new site ./translation/
Congratulations! Your new Hugo site is created in /Users/fuxiaosong/dev/fanyi.

Just a few more steps and you're ready to go:

1. Download a theme into the same-named folder.
   Choose a theme from https://themes.gohugo.io/, or
   create your own with the "hugo new theme <THEMENAME>" command.
2. Perhaps you want to add some content. You can add single files
   with "hugo new <SECTIONNAME>/<FILENAME>.<FORMAT>".
3. Start the built-in live server via "hugo server".

Visit https://gohugo.io/ for quickstart guide and full documentation.
fuxiaosongs-Air:dev fuxiaosong$
```

> 启动服务：

```
hugo server --theme=material-design --buildDrafts
```

> 构建并部署服务：

```
hugo --theme=material-design --baseUrl="https://translation-36518.firebaseapp.com"

firebase deploy --project translation-36518
```
