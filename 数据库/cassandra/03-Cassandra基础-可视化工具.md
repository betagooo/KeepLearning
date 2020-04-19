# IDEA
## 配置连接Cassandra

1. 打开IDEA的侧边栏工具【database】，点击【+】新增数据库，选择【Apache Cassandra】

   ![image-20200408011225509](imageAssets/image-20200408011225509.png)

2. 安装驱动。首次使用，IDEA会提示下载相应驱动，确认就好。

3. 配置Cassandra连接。

   ![image-20200408011036774](imageAssets/image-20200408011036774.png)
   配置完，点击【Test Connection】可以看到是否连接成功。
   
   ![image-20200408012123942](imageAssets/image-20200408012123942.png)

## 简单使用

IDEA中支持以下功能：

![image-20200408012314096](imageAssets/image-20200408012314096.png)

### console

新建一个query console，可以在上面执行cql。

![image-20200408013106890](imageAssets/image-20200408013106890.png)