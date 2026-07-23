# SprintAI Windows x64 发布手册

本文档供维护者创建 GitHub Release 时使用

## 版本与资产命名

- Git 标签：`v0.x.y`，稳定公开版本从 `v1.0.0` 开始
- 安装包：`SprintAI-vX.Y.Z-windows-x64.exe`
- 校验文件：`SprintAI-vX.Y.Z-windows-x64.exe.sha256`

修复问题时递增补丁版本，加入兼容的新能力时递增次版本，出现重大不兼容变化时递增主版本

## 发布前检查

### 1. 自动化验证

在 `D:\产品经理\apps\desktop` 执行：

```powershell
npm run test
npm run lint
npm run build
```

在 `D:\产品经理\apps\backend` 执行：

```powershell
$env:PYTHONPATH='src'
& "D:/Anaconda.python/envs/sprintai_backend/python.exe" -m pytest tests -v
```

所有命令必须成功完成，不能只根据部分日志判断通过

### 2. 真实 DeepSeek 端到端冒烟

1. 使用有效的 DeepSeek 配置启动后端
2. 启动 Windows 桌面端或待发布安装包
3. 从前端创建真实会话
4. 从“定义问题”开始走完整流程，直到生成故事板
5. 确认群聊、白板、流程图、投票、用户拍板和故事板在前端可见
6. 至少验证一次用户打断、阶段检查点和真人专家环节
7. 确认流程结束后会话可以重新打开，关键产出仍然存在

`apps/backend/scripts/smoke_real.py` 只能作为接口级辅助检查，不能替代前端可见的完整冒烟

### 3. Windows 安装检查

在干净的 Windows x64 环境验证：

- 全新安装
- 首次启动
- 覆盖升级
- 正常退出后再次启动
- 卸载
- 开始菜单和桌面快捷方式
- 安装路径中包含中文或空格时仍能运行

### 4. 安全与隐私检查

确认安装包中不包含：

- API Key 或访问令牌
- `.env` 文件
- 本地数据库
- 测试会话或测试账号
- 调试日志中的个人信息
- 源码映射或不需要公开的内部文件

## 生成 SHA-256

在安装包所在目录执行：

```powershell
$file = '.\SprintAI-v0.1.0-windows-x64.exe'
$hash = (Get-FileHash $file -Algorithm SHA256).Hash.ToLower()
"$hash  $(Split-Path $file -Leaf)" | Set-Content -Encoding ascii "$file.sha256"
```

生成：

```text
SprintAI-v0.1.0-windows-x64.exe.sha256
```

再次执行以下命令并确认输出与 `.sha256` 文件一致：

```powershell
Get-FileHash '.\SprintAI-v0.1.0-windows-x64.exe' -Algorithm SHA256
```

## 创建 Release

1. 更新 `CHANGELOG.md`
2. 确认版本号、Git 标签和安装包文件名完全一致
3. 在 GitHub 创建 Draft Release
4. 复制 `docs/RELEASE_TEMPLATE.md` 并替换版本与本次变化
5. 上传 `.exe` 与 `.sha256` 两个资产
6. 检查最新版本、历史版本和反馈链接
7. 预览 Release 页面，确认没有内部信息或未替换内容
8. 发布 Release
9. 从 Releases 页面重新下载安装包
10. 在干净的 Windows x64 环境完成一次最终冒烟

## 发布后

- 确认 `https://github.com/Findworth/sprint/releases/latest` 指向新版本
- 将本次变化同步到 `CHANGELOG.md`
- 观察新建问题，并在修复后更新对应 Issue
- 若发现阻塞性问题，立即撤下有问题的 Release，修复后递增补丁版本重新发布
