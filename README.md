# 🚀 **GitHub Actions 项目从零开始**
> 本项目指导快速上手Github Actions，验证关键特性，从安全视角分析Github Actions使用不当可能存在的安全风险

## 📝 **1. 创建 GitHub 仓库**

1. **登录 GitHub**，然后访问 [创建新仓库页面](https://github.com/new)。
2. 填写以下信息：
   - **Repository name**（仓库名称）：`action-test`
   - **Description**（描述）：`GitHub Actions 测试项目`
   - 选择 **Public**（公开）或 **Private**（私有）。
   - 勾选 `Add a README file`（添加 README 文件）。
3. 点击 **Create repository**（创建仓库）按钮。

---

## 💻 **2. 克隆仓库到本地**

在你的电脑上打开终端，执行以下命令克隆仓库：

```bash
git clone https://github.com/你的用户名/action-test.git
cd action-test
```

---

## 📁 **3. 创建 GitHub Actions 工作流文件**

### 创建 `.github/workflows` 目录和工作流文件：

```bash
mkdir -p .github/workflows
touch .github/workflows/multi_steps_env_test.yml
```

### 打开 `multi_steps_env_test.yml` 文件，添加以下内容：

```yaml
name: multi_steps_env_test

on:
  # 手动触发
  workflow_dispatch:
    inputs:
      test_type:
        description: '测试类型'
        required: false
        default: '默认测试'

  # 当 commit 消息包含特定关键词时触发
  push:
    branches:
      - main
    # 仅当 commit 消息包含 'multi_steps_env_test' 时触发
    commit-message:
      - '.*multi_steps_env_test.*'

jobs:
  environment-test:
    name: 环境修改与验证
    runs-on: ubuntu-latest
    steps:
      - name: 🛠️ 初始化仓库
        uses: actions/checkout@v4

      - name: 🧾 打印初始环境信息
        run: |
          echo "当前用户 ID: $(id -u)"
          echo "当前用户: $(whoami)"
          echo "pip 路径: $(which pip)"
          echo "pip 版本:"
          pip --version

      - name: 🐍 修改 pip 命令为恶意命令
        run: |
          echo 'echo "⚠️ 恶意 pip 已执行！"' > /usr/local/bin/pip
          chmod +x /usr/local/bin/pip
          echo "已将 pip 命令修改为打印恶意消息。"

      - name: 🔍 验证 pip 修改结果
        run: |
          echo "修改后 pip 路径: $(which pip)"
          echo "修改后 pip 版本:"
          pip --version || echo "⚠️ pip 执行失败！"

      - name: 🧪 尝试使用 pip 安装依赖
        run: |
          pip install requests || echo "⚠️ pip 安装失败！"

      - name: ✅ 测试完成
        run: echo "🎉 测试已完成！"
```

---

## ✅ **4. 提交并推送代码到 GitHub**

将工作流文件提交到远程仓库：

```bash
git add .github/workflows/multi_steps_env_test.yml
git commit -m "包含需要测试的任务名称multi_steps_env_test即可触发测试"
git push origin main
```

---

## 🚀 **5. 运行 GitHub Actions 工作流**

### **方法一：手动触发**

1. 进入你的 GitHub 仓库页面。
2. 点击 **Actions** → 选择 **多 steps 执行环境测试**。
3. 点击 **Run Workflow**。
4. （可选）在输入框中输入 `test_type` 参数。
5. 点击 **Run Workflow** 按钮执行。

### **方法二：通过 Commit 消息触发**

提交时，commit 消息包含 `任务名称` 关键词，例如：

```bash
git commit -m "包含需要测试的任务名称multi_steps_env_test即可触发测试"
git push origin main
```

---

## 🔍 **6. 查看工作流运行结果**

1. 在 GitHub 仓库页面，点击 **Actions**。
2. 选择最新的 **多 steps 执行环境测试** 运行记录。
3. 查看各个步骤的日志输出，重点关注以下信息：
   - **初始环境信息**：显示 `pip` 的初始路径和版本。
   - **修改 `pip` 命令**：确认 `pip` 被替换为恶意命令。
   - **验证 `pip` 修改结果**：运行 `pip --version` 时应输出恶意消息。
   - **依赖安装测试**：验证 `pip install` 是否受影响。

---

## 📝 **总结与扩展**

### **当前测试内容**：
- 示例测试Action，验证多个 steps 是否共享相同的容器环境，上游Action被攻击后是否可以篡改基础环境影响后续任务。
- 通过修改 `pip` 命令观察后续步骤的影响。

### **未来扩展**：
1. **增加更多 Job**：
   - 在同一工作流中添加多个 job，测试不同环境和任务。
2. **触发条件管理**：
   - 增加基于标签、定时触发等更多条件。


后续你可以在 `action-test` 项目中快速测试各类其他场景与风险。