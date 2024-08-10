参卡：
https://www.youtube.com/watch?v=H-XBVD1Y8Qs

## 主要步骤

1. 创建Container：
		docker run --detach  --publish 443:443  --publish 8080:80 --publish 1001:22 -v /your_ws/gitlab/etc:/etc/gitlab -v /your_ws/gitlab/log:/var/log/gitlab -v /your_ws/gitlab/opt:/var/opt/gitlab --restart always --shm-size=4gb --name=gitlab gitlab/gitlab-ce:latest
		\

2. 修改root密码 
		docker exec -it gitlab1 grep 'Password:' /etc/gitlab/initial_root_password
	通过上述指令可以活得root用户的密码，然后登陆localhost:8080可以访问gitlab网页
	**Note： 第一次运行gitlab要等5分钟左右**
	

3. 修改root密码 
	  左上角管理员人头像 -> Edit Profile -> 左边栏Password -> 修改密码

4. 添加普通用户
	 - 普通用户一样访问localhost:8080然后注册，需要管理员root用户批准
	 - 管理员登陆界面后选择左上角的Menu菜单，然后选择最下面的admin，左侧则出现很多items，选择Overview
	 - 从Overview中选择Users，则会出现关于User的管理，选择Pending Approval，则会看到普通用户注册的信息，选择approve即可
	
	 	