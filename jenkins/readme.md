## Jenkins 的五大超能力（2018年）
1. **Jenkins Pipeline**：流水线（Pipeline）与Blue Ocean的搭配，让用户轻松驾驭持续交付的核心实践 ---- 流水线
2. **Jenkins Evergreen**：常青树项目，可以帮助我们自动化升级Jenkins Master
3. **Configuration As Code**：配置即代码，可以让用户像管理代码一样管理Jenkins的系统配置
4. **Cloud Native Jenkins**：云原生Jenkins，让Jenkins更适合在公有云与私有云环境下运行
5. **Jenkins X**：面向Kubernetes的云原生应用CI/CD解决方案

## 安装与使用
1. 安装虚机并安装docker
    + 安装centos 7 虚机
        > 新版的Vmare安装centos7后，第一次开机可能需要选择安装步骤，依次键入：1，2，q，yes即可。
    + 安装docker
        ```
        // 先更新软件包和依赖
        yum -y update
        // 清除机器上的docker文件
        sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
        
        // 添加存储库
        sudo yum install -y yum-utils \
             device-mapper-persistent-data \
             lvm2
        
        sudo yum-config-manager \
                --add-repo \
                https://download.docker.com/linux/centos/docker-ce.repo

        // 安装docker
        // 安装最新版
        sudo yum install docker-ce docker-ce-cli containerd.io
        // 安装特定版本
        yum list docker-ce --showduplicates | sort -r // 查看版本
        sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io  // 选择版本安装

        // 检查是否安装成功
        // docekr version 

        // 将仓库变更为国内的镜像仓库
        curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://f1361db2.m.daocloud.io

        // 重启docker
        sudo systemctl start docker

        // 查看镜像仓库是否更换成功
        docker info 
        // 如下为更换成功
        Registry Mirrors:
        http://f1361db2.m.daocloud.io/
        ```

2. 通过docker安装容器
    + 安装jenkins
         ```
         // 官方给的配置如下
         docker run \
            -u root \            ## user
            --rm \               ## 关闭时自动删除Docker容器，这里不建议使用
            -d \                 ## 后台运行容器
            -p 8080:8080 \       ## 暴露/映射端口，使得外部可以访问
            -p 50000:50000 \ 
            -v jenkins-data:/var/jenkins_home \  ## 映射到本地卷，根据需求自己修改映射，:号左边为本机目录，右边为docker容器内部的目录
            -v /var/run/docker.sock:/var/run/docker.sock \ 
            jenkinsci/blueocean  ## 镜像名
        // 这里建议加一个使用 --name参数，给容器起一个别名
         ```
    + 安装Mysql
        1. 安装mysql
            ```
            docker run \
	        -d \
	        -p 3306:3306 \
	        --name mysql \
	        -v /docker/mysql/conf:/etc/mysql/conf.d \
	        -v /docker/mysql/logs:/logs \
	        -v /docker/mysql/data:/var/lib/mysql \
	        -e MYSQL_ROOT_PASSWORD=123456 \
	        mysql:5.7
            ```
        2. 创建数据库和添加远程登录用户名
            ```
            create database sonar;

            CREATE USER 'sonar'@'%' IDENTIFIED WITH mysql_native_password BY 'sonar';
            GRANT ALL PRIVILEGES ON *.* TO 'sonar'@'%';
            ```
    + 安装sonarQube
       1. 安装SonarTest容器
            ```
            // 1.创建/docker/sonarqube文件夹
            mkdir -p /docker/sonarqube

            // 2.创建容器
            docker run -d --name sonartest sonarqube:7.4-community

            // 3.进入容器内部拷贝文件
            docker exec -it sonarqube /bin/bash
            // 命令解释：将conf/ data/ extensions文件递归下拷贝到/docker/sonarqube目录中
            scp -r conf/ data/ extensions/ logs/ root@127.0.0.1:/docker/sonarqube
            scp是ssh的cp，root为用户名，127.0.0.1为远程服务器的密码。
            输入yes，然后输入密码

            // 4.退出容器并删除容器
            exit // 退出容器
            docker stop sonartest // 暂停容器运行
            docker rm sonartest // 删除容器

            // 5.修改文件夹权限
            chmod -R 777 /docker/sonarqube
            ```
        2. 安装sonarqube并于mysql服务建立联系
            ```
            docker run \
            -d \
            --name sonarqube \
            -p 9000:9000 \
            -p 9092:9092 \
            --link=mysql:mysql \
            -v /data/sonarqube/logs:/opt/sonarqube/logs \
            -v /data/sonarqube/conf:/opt/sonarqube/conf \
            -v /data/sonarqube/data:/opt/sonarqube/data \
            -v /data/sonarqube/extensions:/opt/sonarqube/extensions \
            -e SONARQUBE_JDBC_USERNAME=sonar \
            -e SONARQUBE_JDBC_PASSWORD=sonar \
            -e SONARQUBE_JDBC_URL="jdbc:mysql://127.0.0.1:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false" \
            sonarqube:7.4-community
            注意
            ```
    + 部署Sonar-Scanner
    + 参考资料：[mysql+sonar](https://blog.csdn.net/KissedBySnow/article/details/90437605)
        
    
    
3. Jenkins的相关配置
    + Jenkins与Gitlab建立连接
        + 现在Gitlab建立访问令牌，注意：建立好访问令牌之后需要立即保存，不然需要重新建立
        ![](https://ae01.alicdn.com/kf/Hd926dd696cff474d91b87f55f2ce89ffu.png)
        + 在Jenkins上进行相关配置
        ![](https://ae01.alicdn.com/kf/He4fa08c0d6484da7acb8c94d12c8ca2fi.png)
        ![](https://ae01.alicdn.com/kf/H84e0a788811e49178a78e0c44255a4c4I.png)
        ![](https://ae01.alicdn.com/kf/H2cd17a8d242a466f9daecd64f3b534aew.png)
    + Jenkins添加git凭据
        + 添加全局凭据，然后将git的<font color=red>私钥</font>复制上去
        ![](https://ae01.alicdn.com/kf/H3d1c7ec45f724c338dd11d2b3a0b6b7av.png)
    + Jenkins添加slave节点
        + 添加slave节点
        ![](https://ae01.alicdn.com/kf/H89459139210340cd800bab079ec1fa47t.png)
        + slave节点的配置
        ![](https://ae01.alicdn.com/kf/Hb858aaedb36e48b08af6dd030c6d3d85z.png)
        ![](https://ae01.alicdn.com/kf/H023cbb73e9a444ca9f10fe134c66fcb9r.png)

