在使用 Git 时，你可能需要配置多个远程仓库，并且每个仓库可能需要使用不同的 SSH 密钥。以下是如何配置多个远程地址以及对应的 SSH 密钥的步骤：

### 1. 生成 SSH 密钥
首先，为每个远程仓库生成不同的 SSH 密钥对。假设你已经生成了两个密钥对：

- 用于 GitHub 的密钥对：`id_rsa_github` 和 `id_rsa_github.pub`
- 用于内网 Git 的密钥对：`id_rsa_internal` 和 `id_rsa_internal.pub`

```bash
# 生成 GitHub 的 SSH 密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_github

# 生成内网 Git 的 SSH 密钥
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa_internal
```

### 2. 配置 SSH 配置文件
编辑 `~/.ssh/config` 文件，为每个远程仓库配置不同的 SSH 密钥。

```bash
# 编辑 SSH 配置文件
nano ~/.ssh/config
```

在文件中添加以下内容：

```plaintext
# GitHub 配置
Host github.com
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_rsa_github

# 内网 Git 配置
Host internal-git
    HostName internal.git.example.com
    User git
    IdentityFile ~/.ssh/id_rsa_internal
```

### 3. 添加远程仓库
在 Git 项目中，添加多个远程仓库。假设你已经有一个 GitHub 的远程仓库，现在要添加一个内网的远程仓库。

```bash
# 添加 GitHub 远程仓库（如果还没有）
git remote add origin git@github.com:username/repository.git

# 添加内网 Git 远程仓库
git remote add internal git@internal-git:username/repository.git
```

### 4. 验证配置
你可以通过以下命令验证 SSH 配置是否正确：

```bash
# 测试 GitHub 连接
ssh -T git@github.com

# 测试内网 Git 连接
ssh -T git@internal-git
```

如果配置正确，你应该会看到类似以下的输出：

```plaintext
Hi username! You've successfully authenticated, but GitHub does not provide shell access.
```

### 5. 使用不同的远程仓库
现在你可以使用不同的远程仓库进行推送和拉取操作。

```bash
# 推送到 GitHub
git push origin main

# 推送到内网 Git
git push internal main
```

### 6. 查看远程仓库
你可以使用以下命令查看当前配置的远程仓库：

```bash
git remote -v
```

输出应该类似于：

```plaintext
origin  git@github.com:username/repository.git (fetch)
origin  git@github.com:username/repository.git (push)
internal  git@internal-git:username/repository.git (fetch)
internal  git@internal-git:username/repository.git (push)
```

### 总结
通过以上步骤，你可以为不同的远程仓库配置不同的 SSH 密钥，并且轻松地在多个远程仓库之间进行切换和操作。