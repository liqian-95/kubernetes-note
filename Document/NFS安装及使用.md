#### NFS安装
参考来源：
https://help.ubuntu.com/lts/serverguide/network-file-system.html
https://help.ubuntu.com/community/NFSv4Howto
https://www.cnblogs.com/MoreExcellent/p/7222895.html

NFS简介：nfs服务端提供共享目录，客户端可将该目录挂载到本地。在本地该目录下的任意操作，可同步到nfs服务端相应的目录。

服务端ip：192.168.0.127，  客户端ip：192.168.0.137

-   安装服务端
    ```
    sudo apt install nfs-kernel-server
    ```
-   配置nfs服务端
    ```
    ## 编辑/etc/exports，修改内容如下：
    /home *(rw,sync,no_root_squash)
    ### /home   ：共享的目录
    ### *       ：指定哪些用户可以访问
    ###         *  所有可以ping同该主机的用户
    ###         192.168.1.*  指定网段，在该网段中的用户可以挂载
    ###         192.168.1.12 只有该用户能挂载
    ### (ro,sync,no_root_squash)：  权限
    ###         ro : 只读
    ###         rw : 读写
    ###         sync :  同步
    ###         no_root_squash: 不降低root用户的权限
    其他选项man 5 exports 查看
    ```
-   重启nfs服务
    ```
    sudo systemctl start nfs-kernel-server.service
    ```

-   安装客户端
    ```
    sudo apt install nfs-common
    ```
-   查看NFS服务器的共享目录
    ```
    showmount -e 192.168.0.127
    ```
    注：若遇到：clnt_create: RPC: Program not registered，可尝试使用以下方案：
    ```
    ## 查看nfs各种服务，然后重启
    service nfs-* restart
    ```
-   挂载NFS服务器共享目录到本地某一目录
    ```
    sudo mount 192.168.0.127:/home /es/
    ```
    注：挂载点 即当地的目录必须已经存在。而且在该目录中没有文件或子目录。
    
-   NFS服务容器镜像-Dockerfile
    来源：https://github.com/kubernetes/examples/blob/master/staging/volumes/nfs/nfs-data/Dockerfile
    
    ```
    FROM centos
    RUN yum -y install /usr/bin/ps nfs-utils && yum clean all
    RUN mkdir -p /exports
    ADD run_nfs.sh /usr/local/bin/
    ADD index.html /tmp/index.html
    RUN chmod 644 /tmp/index.html
    
    # expose mountd 20048/tcp and nfsd 2049/tcp and rpcbind 111/tcp
    EXPOSE 2049/tcp 20048/tcp 111/tcp 111/udp
    
    ENTRYPOINT ["/usr/local/bin/run_nfs.sh", "/exports"]
    ```
    run_nfs.sh
    ```
    #!/bin/bash
    function start()
    {
        # prepare /etc/exports
        for i in "$@"; do
            # fsid=0: needed for NFSv4
            echo "$i *(rw,fsid=0,insecure,no_root_squash)" >> /etc/exports
            # move index.html to here
            /bin/cp /tmp/index.html $i/
            chmod 644 $i/index.html
            echo "Serving $i"
        done
      
        # start rpcbind if it is not started yet
        /usr/sbin/rpcinfo 127.0.0.1 > /dev/null; s=$?
        if [ $s -ne 0 ]; then
           echo "Starting rpcbind"
           /usr/sbin/rpcbind -w
        fi
    
        mount -t nfsd nfds /proc/fs/nfsd
    
        # -N 4.x: disable NFSv4
        # -V 3: enable NFSv3
        /usr/sbin/rpc.mountd -N 2 -V 3 -N 4 -N 4.1
    
        /usr/sbin/exportfs -r
        # -G 10 to reduce grace time to 10 seconds (the lowest allowed)
        /usr/sbin/rpc.nfsd -G 10 -N 2 -V 3 -N 4 -N 4.1 2
        /usr/sbin/rpc.statd --no-notify
        echo "NFS started"
    }
    
    function stop()
    {
        echo "Stopping NFS"
    
        /usr/sbin/rpc.nfsd 0
        /usr/sbin/exportfs -au
        /usr/sbin/exportfs -f
    
        kill $( pidof rpc.mountd )
        umount /proc/fs/nfsd
        echo > /etc/exports
        exit 0
    }
    
    
    trap stop TERM
    
    start "$@"
    
    # Ugly hack to do nothing and wait for SIGTERM
    while true; do
        sleep 5
    done
    ```