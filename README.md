### 编辑
```bash
hexo new "pagetitle"
# 于目录下：
source/_posts

```

### 部署
```bash
hexo g

hexo d

# 部署配置 _config.yml
deploy:
  type: git
  repo: root@132.232.74.127:blog.git
  branch: master

```

