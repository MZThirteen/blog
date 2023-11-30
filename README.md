## 博客

hexo [官网地址](https://hexo.io/zh-cn/)

### 命令说明

```
// 本地运行
yarn server
// 打包
sh ./build.sh
// 新建文章
hexo new [layout] <title> -p <文件名>
```

### 目录说明

```
├── .deploy_git // 部署之前预先生成的静态文件
├── scaffolds // 模版文件夹
├── source // 资源文件夹是存放用户资源的地方
│   ├── _data // 数据文件夹
│   ├── _posts // post模版的文章
│   ├── assets // 全局资源文件夹
│   │   ├── cover
│   │   └── images
│   ├── categories // 分类页面
│   ├── link // 友情链接页面
│   └── tags // 标签页面
├── themes // 主题文件夹
└── _config.yml // 网站的配置信息
```
