# calculator
牛马计算器 v1.28 — onedir + updater 架构说明
架构变更（v1.28）
从 --onefile 自替换改为 --onedir + 独立 updater.exe：
项目
旧架构（v1.27及以前）
新架构（v1.28）
主程序打包
--onefile（单文件，启动时解压到 _MEIxxxxx）
--onedir（目录，启动快，无解压）
更新方式
PowerShell 脚本自替换
独立 updater.exe（--onefile，体积小）
更新包
单个 exe
update_vX.XX.zip（整个目录的增量文件）
DLL 问题
_MEI 临时释放导致 python314.dll 加载失败
无临时释放，彻底解决
版本信息
version.txt
version.json（本地+远程统一）
发布目录结构
牛马计算器/
├─ 牛马计算器.exe        （主程序，onedir）
├─ updater.exe           （更新器，onefile）
├─ version.json          （本地版本信息）
├─ calculator.ico
├─ python314.dll         （onedir 依赖）
├─ _internal/            （onedir 依赖）
└─ resources/            （资源，可选）
打包步骤（Windows）
双击运行 build_onedir.bat
产物在 dist\牛马计算器\
测试运行 dist\牛马计算器\牛马计算器.exe，点 ⓘ 检查更新（应显示"已是最新版本"）
发布更新步骤
修改代码，版本号 APP_VERSION 改为新版（如 v1.29），version.json 同步
运行 build_onedir.bat 重新打包
将 dist\牛马计算器\` 内**所有文件**压缩为 update_v1.29.zip`
注意：zip 内应直接是文件，不要多一层目录
必须包含：牛马计算器.exe、_internal/、python314.dll 等 onedir 全部文件
建议包含：updater.exe（如果更新器也有改动）、version.json
计算 update_v1.29.zip 的 SHA256：
   certutil -hashfile update_v1.29.zip SHA256
在 Gitee 和 GitHub 创建 Release：
tag: v1.29
附件：update_v1.29.zip
更新 Gitee 和 GitHub 的 version.json（仓库根目录）：
   {"version": "v1.29", "sha256": "<刚才算出的SHA256>"}
更新流程（用户视角）
用户点 ⓘ → 检查更新
发现新版 → 下载 update.zip（断点续传+进度条）
SHA256 校验
启动 updater.exe，主程序退出
updater：等待主程序退出 → 备份 → 解压覆盖 → 校验 → 拉起新版 → 清理
失败自动回滚
updater 保护机制
进程等待：tasklist 确认主程序完全退出，最多等 30 秒
文件锁检测：wait_file_writable 对抗 Defender/火绒/360 扫描锁定，带重试
回滚：备份整个程序目录，任何一步失败自动恢复
Zip Slip 防护：解压时检查路径穿越
彻底脱离拉起：cmd /c start 拉起新版，避免子进程上下文问题
