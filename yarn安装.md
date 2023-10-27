1.安装yarn
	npm install -g yarn
2.yarn换源
	查看yarn配置
		yarn config get registry
		/
		yarn config list
	更换镜像
		全局修改
			yarn config set registry https://registry.npm.taobao.org
		临时修改
			yarn add 软件名 --registry https://registry.npm.taobao.org/
		使用yrm切换镜像源
			安装yrm
				npm install -g yrm
			查看可用镜像源
				yrm ls
			测试镜像源访问速度
				yrm test 镜像源
			切换镜像源
				yrm use 镜像源
