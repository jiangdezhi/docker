# 将已知镜像上传到DockerHub
##1 标记已知镜像
docker tag <existing image> <hub-user>/<repo-name>[:<tag>]
这里的tag不指定就是latest

##2 docker hub 帐号在本地验证登陆

##3 docker push 镜像到docker hub 的仓库
docker push<hub-user>/<repo-name>:<tag>

##4 验证一下
