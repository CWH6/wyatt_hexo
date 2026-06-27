# Wyatt's Blog  

基于 [Hexo](https://hexo.io/) + [Butterfly](https://butterfly.js.org/) 主题的个人博客。

## 环境要求

- Node.js >= 18    
- npm

## 快速开始

```bash
# 安装依赖
npm install

# 本地启动预览
npm run server
# 或
npx hexo s

# 访问 http://localhost:4000
```

## 常用命令

| 命令 | 说明 |
|------|------|
| `npm run server` | 本地启动预览 |
| `npm run build` | 生成静态文件到 `public/` |
| `npm run clean` | 清缓存 |
| `npx hexo g` | 生成静态文件（等同于 build） |
| `npx hexo clean && npx hexo g` | 清缓存并重新生成 |

### 新建文章

```bash
npx hexo new "【分类】文章标题"
```

例如：

```bash
npx hexo new "【文档】Git 使用指南" 
npx hexo new "【生活】爬山日记"
```

文章文件会生成在 `source/_posts/` 目录下。

### 重新生成（修改配置后）

```bash
npm run clean && npm run build
```

## 项目结构

```
wyatt_hexo/
├── _config.yml              # Hexo 主配置
├── _config.butterfly.yml    # Butterfly 主题配置
├── source/
│   ├── _posts/              # 文章目录
│   ├── about/               # 关于页面
│   ├── works/               # 作品页面
│   └── img/                 # 图片资源
├── themes/
│   └── butterfly/           # Butterfly 主题
└── public/                  # 生成的静态站点（部署用）
```

## 部署

将 `public/` 目录下的文件上传到服务器或 Pages 服务即可。   
