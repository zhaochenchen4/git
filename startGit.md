一、本地绑定git
	1.使用Git Bash 进入窗口 执行指令查看是否安装SSH
		ssh
	2.执行指令，回车生成RSA密钥 
		ssh-keygen -t rsa
	  Windows生成目录当前用户下的.ssh文件中
	  Linux ~/.ssh
	3.将生成的id_rsa.pub文件上传到git设置SSH and GPGkeys
	4.验证是否绑定成功
		ssh -T git@github.com
	  提示successfully即可
二、创建仓库本地拉取和提交
1.git中创建自己的仓库
2.复制当前仓库的https地址
3.在本地文件夹中进入Git Bash初始化git 
	git init
4.执行命令克隆仓库  
	git clone https://...
5.进入本地克隆的文件夹可以看到仓库的文件
6.执行指令查看仓库状态
	git status
7.若存在新文件但未加入git的会出现红字提示
8.执行指令添加新文件到git
	git add .(全部文件)|| 文件名/ (某文件全部文件)
9.执行指令提交文件
	git commit -m "提交注释"
10.查看仓库日志
	git log
11.执行指令推送已提交文件
	git push origin 分支名

同步远程仓库和本地仓库
	git pull origin 分支
本地已有仓库，关联远程仓库指令
	git remote add 远程仓库名字 仓库地址

fatal: unable to access "xxx": OpenSSL SSL_read:Connection was reset, errno 10054问题：
禁用证书
	git config --global http.sslVerify false
	检查git的username email是否与本地配置的一致，若不一致修改本地配置：
	git config user.name
	git config --global user.name "username"
	git config --global user.email  "email"
	||
	git config --replace-all user.name "username"
	git config --replace-all user.email "email"

Failed to connect to 127.0.0.1 port 1080: Connection refused问题:
	查询动态代理
	git config --global http.proxy
	git config --global https.proxy
	有返回值，取消代理
	git config --global --unset http.proxy
	git config --global --unset httpx.proxy
	将C盘用户下的.gitconfig文件中的proxy行删掉
git提交报 husky - pre-commit hook exited with code 1 (error)	
	git commit --no-verify -m 'xxxxxx'