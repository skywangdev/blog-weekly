---
date: 2022/03/19
---

<img src="https://data.skywangdev.com/blog/S-6.jpeg" width="800" />


* * *

![](https://images.yasking.org/technology/1683194581/08.jpg)

Ansible 是一种开源自动化工具，用于自动化配置管理、应用程序部署和云基础设施部署等任务。它使用 SSH 协议通信，不需要在远程主机上安装额外的软件。

### Hello World！

```bash
$ sudo apt install ansible
```

配置文件所在路径 */etc/ansible/ansible.cfg* 和 */etc/ansible/hosts*，前者存储配置，后者存储被管理的机器信息（Inventory）。

添加本机为被管理机器

```bash
$ /bin/echo -e "[local]\nlocalhost ansible_connection=local" >> /etc/ansible/hosts
```

执行

```bash
$ ansible localhost -m command -a "echo Hello World."
```

![](https://images.yasking.org/technology/1683194581/01.jpg)

### 添加一台远程服务器

这是一台远程服务器，将机器配置到 */etc/ansible/hosts*

![](https://images.yasking.org/technology/1683194581/06.jpg)

另外也需要修改配置 */etc/ansible/ansible.cfg*

```yaml
[defaults]
private_key_file = /root/.ssh/id_rsa
host_key_checking = False
```

设置默认的 ssh 私钥地址和首次登录时不进行校验

执行

```bash
$ ansible or01 -m ping
```

![](https://images.yasking.org/technology/1683194581/07.jpg)

### Ansible 的使用方式

> 有两种使用 Ansible 的方式：**Ad-Hoc command** 和 **Playbook**，前者是通过一次次简短的指令來操作 Ansible，而后者则是先把任务写好，然后再一次执行。两者的关系就好比我们在 Linux Shell 里打指令和先写个 Shell Script 再执行一样。

上例输出 “Hello World.” 即一次简短的指令，又比如：`ansible all -m ping`

![](https://images.yasking.org/technology/1683194581/02.jpg)

> Playbook 就字面上的意思為剧本。我們可以透过事先写好的剧本 (Playbooks) 來让各个 Managed Node 进行指定的动作 (Plays) 和任务 (Tasks)。

Playbooks 有许多优点，例如使用 YAML 格式，写配置就像写 Code，比 Shell Script 更具表达能力，支持多种语法。

### 编写并执行 Playbooks

在一份 Playbook 中，可以有多个 Play、多个 Task 和多个 Module

+   Play 通常表示某个特定的目的，例如：“部署服务”，“重启 API 网关”
+   Task 是要完成 Play 所需的具体步骤，例如：“安装 Nginx”，“重启特定的 Docker 容器”
+   Module 是 Ansible 所提供的各种操作方法，例如：`apt: name=vim state=present`（使用 apt 套件安裝 vim）、`command: /sbin/shutdown -r now`（使用 shutdown 的指令重新开机）

示例 Playbook 配置

```yaml
- name: say 'hello world'
  hosts: all
  tasks:
    - name: echo 'hello world'
      command: echo 'hello world'
      register: result
    - name: print stdout
      debug:
        msg: "{{ result.stdout }}"
```

执行

```bash
$ ansible-playbook test.yaml
```

![](https://images.yasking.org/technology/1683194581/03.jpg)

*or01 机器未配置登录方式，演示机器操作失败的情况*

其中，`TASK [Gathering Facts]` 是 Ansible 自动生成的 Task

> 在 Ansible 中，"Gathering Facts" 是指在执行 playbook 之前，Ansible "setup" 模块会自动连接到目标主机并收集关于主机的信息，例如主机的操作系统、IP 地址、CPU 和内存信息等。这些信息可以在 playbook 的后续任务中使用，例如根据主机的操作系统类型执行不同的操作或配置文件。

在指定机器运行 Playbook

```bash
$ ansible-playbook test.yaml -i /etc/ansible/hosts --limit localhost
```

### Ansible 模块 Modules

这是根据类别分类的 Modules 文档：https://docs.ansible.com/ansible/2.9/modules/modules\_by\_category.html

![](https://images.yasking.org/technology/1683194581/04.jpg)

以 Commands 为例，点击进入，并再次点击 “Command”

![](https://images.yasking.org/technology/1683194581/05.jpg)

可以查看相应内容了解模块的使用，Synopsis（简介）、Parameters（参数）、Examples（示例）等

Commond - 执行命令

```yaml
- name: return motd to registered var
  command: cat /etc/motd
  register: mymotd

- name: Run command if /path/to/database does not exist (without 'args' keyword).
  command: /usr/bin/make_database.sh db_user db_name creates=/path/to/database

# 'args' is a task keyword, passed at the same level as the module
- name: Run command if /path/to/database does not exist (with 'args' keyword).
  command: /usr/bin/make_database.sh db_user db_name
  args:
    creates: /path/to/database
```

archive - 解压缩

```yaml
- name: Extract foo.tgz into /var/lib/foo
  unarchive:
    src: foo.tgz
    dest: /var/lib/foo

- name: Unarchive a file that is already on the remote machine
  unarchive:
    src: /tmp/foo.zip
    dest: /usr/local/bin
    remote_src: yes

- name: Unarchive a file that needs to be downloaded (added in 2.0)
  unarchive:
    src: https://example.com/example.zip
    dest: /usr/local/bin
    remote_src: yes
```

粗略看了下，有很多 Module，常用的有以下几种：

+   `apt`安装卸载软件
+   `command` 执行命令
+   `copy` 复制本地文件到远端
+   `file` 管理文件、目录、软链接
+   `lineinfile` 类似 linux 的 sed 指令
+   `service` 管理服务启动停止等
+   `shell` 使用远端的 `/bin/sh` 执行命令，支持变量和管道操作符
+   `stat` 检查档案状态，判断文件是否存在等

### 获取被控端服务器信息

```bash
ansible or01 -m setup | less
```

输出内容太多，执行以下命令输出服务器发行版信息

```bash
$ ansible or01 -m setup -a "filter=ansible_distribution*"
```

![](https://images.yasking.org/technology/1683194581/09.jpg)

本机 Ubuntu

![](https://images.yasking.org/technology/1683194581/10.jpg)

通过 *ansible\_pkg\_mgr* 查询包管理器

```bash
$ ansible all -m setup -a "filter=ansible_pkg_mgr"
```

![](https://images.yasking.org/technology/1683194581/11.jpg)

一个根据发行版信息安装 vim 的配置示例，不匹配的条件将被自动跳过

```yaml
- name: Setup the vim 
  hosts: all
  become: true
  tasks:

    # Debian, Ubuntu.
    - name: install apt packages
      apt: name=vim state=present
      when: ansible_pkg_mgr == "apt"

    # CentOS.
    - name: install yum packages
      yum: name=vim-minimal state=present
      when: ansible_pkg_mgr == "yum"
```

### 使用模版

*hello.txt.j2* - 模版基于 Jinja2

```bash
Hello, {{ dynamic_name }}
```

*test2.yaml*

```yaml
- name: Play the template module
  hosts: localhost
  vars:
    ansible_python_interpreter: /usr/bin/python3
    dynamic_name: "David"

  tasks:
    - name: generation the hello.txt file
      template:
        src: hello.txt.j2
        dest: /tmp/hello.txt

    - name: show file context
      command: cat /tmp/hello.txt
      register: result

    - name: print stdout
      debug:
        msg: "{{ result.stdout_lines }}"
```

输出

![](https://images.yasking.org/technology/1683194581/12.jpg)

注意这里的 `result.stdout_lines`，当使用 register 将内容存储到变量后，该变量有以下几个可选属性

+   stdout：命令的标准输出。
+   stderr：命令的标准错误输出。
+   stdout\_lines：命令的标准输出，按行分割为一个列表。
+   stderr\_lines：命令的标准错误输出，按行分割为一个列表。
+   rc：命令的返回码。
+   start：命令开始执行的时间。
+   end：命令执行结束的时间。
+   delta：命令执行的时间差。

另外，在执行命令时，可以使用 `-e` 参数设置变量覆盖掉配置中预定义的变量

```bash
$ ansible-playbook test2.yaml -e "dynamic_name=Linus"
```

**根据变量引入不同的配置文件**

借助 `var_files` 与模版，参考：[怎麼讓 Playbooks 切換不同的環境？](https://github.com/chusiang/automate-with-ansible/blob/master/14.how-to-use-the-ansible-template-system.md#%E6%80%8E%E9%BA%BC%E8%AE%93-playbooks-%E5%88%87%E6%8F%9B%E4%B8%8D%E5%90%8C%E7%9A%84%E7%92%B0%E5%A2%83)

另外，也建议使用 `group_vars` 和 `host_vars` 切换环境。

**Handler 是什么？怎么使用**

> 在 Ansible 中，handlers 是一种特殊的任务，它们在 playbook 中通常用于在任务执行完成后触发某些操作。handlers 与其他任务的区别在于，它们只有在被通知后才会执行，而不是在每次任务执行时都会运行。

**特点**

+   只有在被任务 notify 的时候才触发执行
+   会等到对应的 play 完成任务后再执行（play 包含 tasks, tasks 负责具体事项的执行）
+   可以保证一个 handler 在一个 play 中只执行一次
+   一个 task 也可以调用多个 handlers
+   handlers 可以像普通任务一样使用条件判断，这样可以根据特定的条件来决定是否执行。

**示例**

```yaml
- hosts: webservers
  tasks:
    - name: task 1
      command: something
      notify: run this once

    - name: task 2 
      command: something else
      notify: 
      - run this once
      - run this again

  handlers:
    - name: run this once
      command: will run only once  
    - name: run this again
      command: will run again
```

另外，在教程中提及了 `post_tasks`，它是什么时候执行呢？顺序如下

![](https://images.yasking.org/technology/1683194581/13.jpg)

另外提到 `handlers` 就必须知晓一个概念，即任务状态，它有三个状态 *changed*、*ok*、*failed*，只有当任务为 changed 状态时，才会触发事件。

任务状态为 changed 的时机

> 需要安装/更新软件包时: 如果需要安装新的软件,或者更新已安装的软件到新版本,则 task 状态会变为 changed
> 
> 需要修改/编辑文件时: 如果修改或编辑了目标主机的配置文件, 则 task 状态会变为 changed
> 
> 首次执行某项任务时: 即使没有带来明显变更,但首次运行一个task, 目标主机的状态已经发生变化, 状态也会变为changed
> 
> 在 idempotent 任务中: 对 idempotent 任务来说, 不管运行多少次, 状态都不变。但它首次运行的时候仍会显示changed状态

### Loop 循环

较早版本的循环借助 `with_items`，示例代码如下

*loop-with-items.yaml*

```yaml
- hosts: localhost
  tasks:  
    - name: Install packages
      apt:
        name: "{{ item }}"
        state: present
      with_items:
         - vim
         - emacs
```

```bash
$ ansible-playbook loop-with-items.yaml -i /etc/ansible/hosts --limit local
```

![](https://images.yasking.org/technology/1683194581/14.jpg)

使用 loop 的实现，同时使用 `label` 改变了临时变量值

*loop-label.yaml*

```yaml
- hosts: localhost
  tasks:
    - name: Install packages
      apt:
        name: "{{ item }}"
      loop: ["vim", "emacs"]
      loop_control:
        label: "Package {{ item }} installation"
```

![](https://images.yasking.org/technology/1683194581/15.jpg)

另外，在以上代码中 {{ item }} 变量是特殊的，不能修改为 {{ item2 }}，从 2.4 版本引入 loop 后，应该使用 loop 来循环，这是更推荐和标准的写法。

loop 提供了不同的选项，除去 label 标签的用法，还有以下功能，在使用时再了解即可。

+   可以控制循环行为，如首次/每次/最后一次循环时执行的动作
+   可以限制循环次数和间隔时间等

本篇暂到这里，接下来学习项目的组织与 Roles

### 参考

+   [現代 IT 人一定要知道的 Ansible 自動化組態技巧](https://github.com/chusiang/automate-with-ansible)
+   [12\. 常用的 Ansible Module 有哪些？](https://github.com/chusiang/automate-with-ansible/blob/master/12.which-are-the-commonly-used-modules.md)