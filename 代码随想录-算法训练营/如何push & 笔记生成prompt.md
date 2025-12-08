**模板说明：**

1. **笔记结构**：
    
    - **题目标题**：使用简洁的标题，并配上适当的 Emoji（如 🔍、🧠）。
    - **题目链接**：提供题目链接，使用 🌐 Emoji。
    - **解题思路**：用 🧠 强调思考过程，解释算法步骤。每个步骤尽量加上清晰的 Emoji 标识，如 📋（步骤），✅（找到目标），🔙（左移），🔜（右移）等。
    - **时间复杂度**：用 ⏱️ 来标示时间复杂度。
    - **代码实现**：代码保持清晰，并加注释。
    - **关键点**：列出代码中的重要概念，标明 O(log n) 复杂度等关键信息，配上 🔑 Emoji。
    - **边界情况**：考虑可能的边界情况，用 🚨、🛑、🚫 表示特殊条件。
    - **总结**：强调算法的效率和优点，用 📚 Emoji。
2. **使用提示**：
    
    - 对话中提供解题代码时，自动生成带有适当 Emoji 的笔记格式。
    - 标注出时间复杂度、边界情况等，尽量使内容条理清晰，易于阅读。


# Git 快速笔记：add / commit / push 操作指南

本笔记用于快速回顾 Git 在本地记录和推送 Obsidian 笔记的基本流程，适合日常同步到 GitHub 使用。

---

## 1️⃣ 初始化仓库（仅第一次）

```bash
# 进入你的笔记文件夹
cd /d/Workshop/obsidian-vault

# 初始化仓库
git init

# 添加文件并提交
git add .
git commit -m "init notes"

# 连接远程仓库（替换为你的 GitHub 地址）
git remote add origin https://github.com/你的用户名/笔记仓.git
git branch -M main
git push -u origin main
```

---

## 2️⃣ 日常更新笔记（每天写完之后）

```bash
# 添加所有变动文件
git add .

# 提交并写提交信息
git commit -m "update notes"

# 推送到 GitHub
git push
```

---

## 3️⃣ 常见问题

🔸 **提示输入用户名和密码？**  
→ GitHub 不支持密码推送，请使用 Personal Access Token（PAT）粘贴作为密码。

🔸 **警告 "LF will be replaced by CRLF"？**  
→ 只是行尾格式提醒，不影响使用。可配置：
```bash
git config --global core.autocrlf true
```

🔸 **要忽略 Obsidian 的本地缓存文件？**  
→ 在仓库根目录建 `.gitignore` 文件，加入：

```
.obsidian/workspace*
.obsidian/cache/
```

---

## ✅ 小建议

- 每天写完就 commit 一次，方便版本管理。
- commit 信息写清楚，比如 "刷题记录 12-08"、"Kaggle Titanic 项目初稿"。
- GitHub 页面可以直接当作学习日志页面发给老师。
