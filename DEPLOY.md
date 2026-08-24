# 部署指南（GitHub Pages）

这份指南教你把 `frontend/` 里的网页部署成一个真正能访问的网站,全程在网页上点点点就能完成,不需要懂命令行。

网站上线后大概长这样:一个 Melbourne 地图,可以按"有没有无障碍卫生间 / 有没有坡道 / 有没有座位 / 有没有停车位"筛选,点每个地点能看到详细信息。

---

## 开始之前:先确认一件事——仓库是公开(Public)还是私有(Private)

GitHub 的免费个人账号,**只有公开仓库才能免费用 GitHub Pages**。私有仓库要用 Pages 的话,需要 GitHub Pro / Team 付费版(如果你是学生,大学邮箱申请 [GitHub Student Developer Pack](https://education.github.com/pack) 可以免费拿到 Pro)。

所以先选一个:

- **方案 A(推荐,最简单)**:把仓库改成 Public。数据库和代码本来就是课程作业,不涉及隐私信息,改成公开最省事。
  仓库页面 → `Settings` → 最下面 `Danger Zone` → `Change repository visibility` → 选 `Public`。
- **方案 B**:保持 Private,但需要仓库属于一个开通了 Pro / Team 的账号。

下面的步骤两种都适用,只是方案 B 需要你确认过账号权限。

---

## 第一步:把这些文件放进仓库

这次新增/修改的文件是:

```
export_frontend_data.py          ← 新增:把数据库导出成网页能读的 JSON
frontend/                         ← 新增:整个网页前端
  index.html
  assets/styles.css
  assets/app.js
  data/*.json                     ← 已经导出好了,不用你重新生成
.github/workflows/deploy-pages.yml ← 新增:自动部署脚本
README.md                         ← 更新:加了前端说明
DEPLOY.md                         ← 就是这份文件
```

### 如果你用网页直接上传(不用装 Git,推荐给不熟悉命令行的同学)

1. 打开你的仓库页面,确认当前在 `Lin-Ma` 分支(或者你们最终要合并的分支)。
2. 点 `Add file` → `Upload files`。
3. 把上面列出的文件夹和文件,从你电脑上**整个拖进浏览器**(现在 GitHub 支持直接拖文件夹,会保留目录结构)。
   - 一定要保证 `frontend` 文件夹、`.github/workflows` 文件夹的层级是对的,不要拖散了。
4. 拖完确认一下左侧文件列表,应该能看到 `frontend/index.html`、`.github/workflows/deploy-pages.yml` 这些路径。
5. 页面下方填一句提交说明,例如 `Add frontend and Pages deployment`,点绿色的 `Commit changes`。

### 如果你用 GitHub Desktop 或命令行

把这些文件复制进本地仓库对应位置,和平时一样 commit + push 到远程分支即可。

---

## 第二步:打开 GitHub Pages 开关

1. 仓库页面 → `Settings` → 左侧菜单找到 `Pages`。
2. 在 **Build and deployment → Source** 这一栏,选择 **`GitHub Actions`**(不要选 "Deploy from a branch",我们用的是自动化脚本部署)。
3. 不用再填别的,保存即可。

---

## 第三步:合并到 main 分支,触发自动部署

部署脚本设置成"只要 `main` 分支有新提交就自动运行"。所以:

1. 把你所在的分支(比如 `Lin-Ma`)发起一个 Pull Request,合并进 `main`。
2. 合并完成后,仓库页面顶部点 `Actions` 标签,应该能看到一个叫 **"Deploy frontend to GitHub Pages"** 的流程正在跑(黄色圆点 = 进行中,绿色对勾 = 成功)。
3. 第一次跑大概 30 秒到 1 分钟。如果不放心,也可以在 `Actions` 页面手动点 `Run workflow` 触发一次。

> 如果暂时不想合并到 main,也可以把工作流文件里 `branches: [ "main" ]` 改成你当前的分支名,先自己测试。

---

## 第四步:打开你的网站

跑完之后,回到 `Settings → Pages`,顶部会出现一行绿字,类似:

```
Your site is live at https://lilylin-star.github.io/MAST90107-accessibility-map/
```

点开就是了。如果显示 404,通常是因为:
- Actions 还没跑完,等一下刷新;
- 仓库还是 Private 又没有 Pro,回到最开始那一步处理;
- Pages 的 Source 选成了 "Deploy from a branch" 而不是 "GitHub Actions"。

---

## 以后数据更新了怎么办?

如果数据库 `database/accessibility.sqlite` 有更新(比如跑了新一轮 `python src/pipeline/build.py`):

1. 本地运行一次:
   ```bash
   python3 export_frontend_data.py
   ```
   这会重新生成 `frontend/data/` 里的几个 JSON 文件。
2. 把更新后的 `database/accessibility.sqlite` 和 `frontend/data/*.json` 一起提交、推到 `main`。
3. Actions 会自动重新部署,网站几分钟内就是最新数据了。

就算你忘了在本地跑这一步也没关系——部署脚本每次都会在 GitHub 的服务器上重新跑一遍 `export_frontend_data.py`,所以网站上的数据永远和仓库里的 `accessibility.sqlite` 保持一致。

---

## 本地先预览一下(可选)

正式部署前,可以在自己电脑上先看效果:

```bash
cd frontend
python3 -m http.server 8000
```

然后浏览器打开 `http://localhost:8000`。**不要**直接双击 `index.html` 打开——浏览器会拦截本地文件的读取请求,地图会一直卡在加载界面。

---

## 常见问题

**Q: 地图一直显示"Loading Melbourne accessibility data…"不动?**
A: 90% 是用双击打开 `index.html` 导致的,按上面"本地先预览一下"用 `http.server` 打开。部署到 Pages 之后不会有这个问题。

**Q: Actions 跑失败了,红叉?**
A: 点进那次运行看日志。最常见的原因是 `database/accessibility.sqlite` 没有一起提交上去——`export_frontend_data.py` 需要这个文件才能生成数据。

**Q: 想要一个更好记的网址?**
A: `Settings → Pages → Custom domain` 可以绑定自己的域名,课程作业一般用不到,提一下备查。
