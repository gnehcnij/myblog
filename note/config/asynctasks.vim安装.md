## Shell配置

### 一、安装asynctasks.vim

#### 1、下载

```
git clone https://github.com/skywind3000/asynctasks.vim.git
```

#### 2、在~/.bashrc中添加互环境变量

```
# asynctasks
export PATH=/opt/asynctasks.vim/bin:$PATH
```

#### 3、在~/.bashrc中设置别名

```
alias t='asynctask -f'
```

#### 4、在~/.config/asynctask/task.ini设置全局任务配置

```
[git-push-master]
command=git push origin master

[git-pull-master]
command=git pull origin master

[git-fetch-master]
command=git fetch origin master

[git-checkout]
command=git checkout $(?branch)

[git-status]
command=git status

[git-log]
command=tig

[misc-disk-usage]
command=df -h

[net-check-port]
command=sudo lsof -i :$(?port)

[update-alternatives list java]
command=sudo update-alternatives --list java

[update-alternatives config java]
command=sudo update-alternatives --config java
```

##### 5、需要fzf的支持，下载fzf

```
git clone https://github.com/junegunn/fzf.git
```

#### 6、安装fzf

```
cd fzf安装目录
./install
```

