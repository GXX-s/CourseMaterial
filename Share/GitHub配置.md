**目录：**

[TOC]

# $\rm I$. `GitHub` 配置

内容不涉及注册 GitHub 的账户，主要**关注点** 是注册好用户名之后配置，包括的内容：

1. 使用 ssh-keygen 生成 ssh key
2. 将 private key 添加到本地 ssh-agent
3. GitHub 配置 public key 

```bash
<<'COMMENT'
最好是在 .ssh 文件夹（MacOS 下的默认位置是 ~/.ssh，Linux 是在 /home/<username>/.ssh，Windows 是在 /c/Users/<username>/.ssh）下进行操作。需要完成的操作如下：
1. 如有必要，需要生成一个 SSH key
2. 确认 SSH 服务已经启动
3. 将 SSH private key 添加到 ssh-agent
以上过程需要在本地完成操作
COMMENT

# 使用 ssh-keygen 来生成 ssh key，命令格式如下：
# -t 需要添加 key 类型，例如 rsa, ed25519, ecdsa
# -f 申明将密码存储在特定的文件下，确认文件名称
# -b 申明 key 的位数大小
# -C 添加一个注释
# ssh-kegen -t filetype -f ssh_key_name -b number_bits -C comment
$ ssh-keygen -t rsa -f rsa_test -b 4096 -C "test@test_mail.com"

Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): # 可选择是否需要输入
Enter same passphrase again:
Your identification has been saved in rsa_test.
Your public key has been saved in rsa_test.pub.
The key fingerprint is:
SHA256:2bcxd5TDwI01C3Iujk080XLdAlwSAberMVLv+3BkZxQ test@test_mail.com
The key's randomart image is:
+---[RSA 4096]----+
|          B*O*++.|
|         .oX ++oo|
|        .o*o+ +E.|
|       . o+* .. .|
|        S .+o   .|
|            +*+o |
|            o*=o.|
|            ..+o.|
|            .... |
+----[SHA256]-----+

# 接下来第二步，测试 ssh-agent 是否已经启动。如果提示了 pid 值那么表示已经启动
$ eval "$(ssh-agent -s)"

Agent pid 49547

# 添加 private key 到 ssh-agent
$ ssh-add -K ~/.ssh/rsa_test

Identity added: /Users/new_user/.ssh/rsa_test (/Users/new_user/.ssh/rsa_test)
```

通过以上步骤生成了 ssh-key [^1][^2]，同时将 private key 添加到了 ssh-agent 中。接下来需要在 GitHub 中添加 public key[^3]。

```bash
<<'COMMENT'
接下来完成配置 GitHub 中的 public key
1. 复制 public key
2. 在 GitHub 中个人界面下 Setting 中去添加 public key
COMMENT

# 复制 private key 对应的 public key（扩展名为 .pub）
$ pbcopy < ~/.ssh/rsa_test.pub

# 将数据黏贴至 GitHub 个人页面下的 Setting 中进行配置
```

在页面 `https://github.com/settings/keys` 完成配置：

![image-20181211170149496](../img/github_setting.png)

在设置的过程，需要添加标题和黏贴 public key

![image-20181211170604883](../img/public_key.png)

**注意事项:**

此外，需要注意⚠️上面命令主要的 Mac OS 系统中，部分命令存在差异——例如，pbcopy 无法在 Linux 和 Windows 中使用。具体的系统命令，可以通过选择不同按钮查看：

![image-20181211173031572](../img/system_command.png)

# $\rm II$. GitHub 多用户设置

前面的配置能够解决单一用户的设置问题，当遇见需要多用户切换使用时（这种情况可能包括两个 GitHub 账户或者 GitHub 与其他托管平台账户的之间），上面的情况就不能解决问题。如果需要解决这个问题，需要通过以下步骤来完成：

1. 配置 config 文件，需要分别设置 Host
2. 仓库配置管理

## 2.1 配置 config

在多用户的情况下，不能使用全局的用户信息和邮箱信息，需要取消相关信息：

```bash
$ git config --global --unset user.name
$ git config --global --unset user.email
```

可以不使用 `--global` 参数进行当前“局部”配置用户信息：

```bash
$ git config user.name "testuser"
$ git config user.email "test@test_mail.com"
```

这里假设我们已经生成了两个 ssh key，其中 private key 的文件是 `~/.ssh/id_rsa` 和 `~/.ssh/rsa_test`；此外对应的 public key 已经在相应的托管平台配置完成。接下来需要在 `~/.ssh` 文件夹下创建一个 `config` 文件（文件名称就是 `config`），并且进行相应的配置，配置内容如下：

```text
# Default GitHub User
Host default	# 可以任意取一个名称
	# User 和 HostName 会生成一个 git@github.com 形式的文件——默认是 git@github.com 可以直连
	# 为了避免错误这里将其拆分为 git 和 github.com
	User git
    HostName github.com
    IdentityFile ~/.ssh/id_rsa	# 申明 default 的 private key 位置
# Second GitHub User
Host second
    User git
    HostName github.com
    IdentityFile ~/.ssh/rsa_test	# 第二个 private key
```

在完成以上配置之后，可以对 Host 进行测试链接情况：

```bash
$ ssh -T default

$ ssh -T second
```

当出现 `Hi <username>! You've successfully authenticated, but GitHub does not provide shell access.` ，表示可以链接。其中 `<username>` 是对应 private key 的用户名。

## 2.2 仓库配置

一般情况下，通过 GitHub 创建仓库之后会出现相应的提示信息，直接输入相应的命令即可完成本地仓库配置，同时也能够进行远程管理。相应的提示信息如下

```bash
$ echo "# test" >> README.md
$ git init	# 初始化本地 repo
$ git add README.md
$ git commit -m "first commit"
$ git remote add origin https://github.com/username/test.git	# 添加 remote
$ git push -u origin master		# 第一次 push
```

上面是一般的情况下，现在需要将部分内容调整，主要是远程 repo 的地址信息需要调整：

```bash
$ git remote add origin second:<username>/test.git
```

其他部分没有相应的调整，操作相同。



# $\rm A$. 参考

[^1]: [Generating a new SSH key and adding it to the ssh-agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/#adding-your-ssh-key-to-the-ssh-agent)   
[^2]: [ssh-keygen(1) - Linux manual page](http://man7.org/linux/man-pages/man1/ssh-keygen.1.html)   
[^3]: [Adding a new SSH key to your GitHub account - User Documentation](https://help.github.com/articles/adding-a-new-ssh-key-to-your-github-account/)  
[^4]: [Mac下配置多个Git账户 ](https://segmentfault.com/a/1190000016269686) 