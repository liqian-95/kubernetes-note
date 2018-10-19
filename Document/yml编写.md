此编写部分未手动操作，仅作参考
注：以version: '2'为参考
-   build
```
build: /path/to/build/dir-Dockerfile
```
用于构建镜像，不常用
-   command
```
command: bundle exec thin -p 3000
或command: [bundle, exec, thin, -p, 3000]
```
使用 command 可以覆盖容器启动后默认执行的命令
-   container_name
```
container_name: app
```
 Compose 的容器名称格式是：<项目名称><服务名称><序号>
虽然可以自定义项目名称、服务名称，但是此命令指定容器的命名
-   depends_on
```
version: '2'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```
通过此命令指定容器启动顺序，例如下面容器会先启动 redis 和 db 两个服务，最后才启动 web 服务
-   tmpfs
```
tmpfs: /run
tmpfs:
  - /run
  - /tmp
```
挂载临时目录到容器内部，与 run 的参数一样效果
-   environment
```
environment:
  - RACK_ENV=development
  - SHOW=true
  - SESSION_SECRET
```
设置镜像变量，它可以保存变量到镜像里面，也就是说启动的容器也会包含这些变量设置
-   env_file
```
env_file:
  - ./common.env
  - ./apps/web.env
  - /opt/secrets.env
```
指定了配置文件， env_file 中路径为配置文件路径。
-   logging
```
logging:
  driver: syslog
  options:
    syslog-address: "tcp://192.168.0.42:123"
```
用于配置日志服务，默认的driver是json-file
-   external_links
```
external_links:
 - redis_1
 - project_db_1:mysql
 - project_db_1:postgresql
```
链接外部容器或其他服务中的容器
-   links
```
links:
 - db
 - db:database
 - redis
```
用于连接同一服务下的某一容器
-   extra_hosts
```
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```
添加主机标签，向/etc/hosts文件中添加，即作ip nane映射
-   ports
```
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"
 - "127.0.0.1:8001:8001"
```
使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口
-   labels
```
labels:
  - "com.example.description=Accounting webapp"
  - "com.example.department=Finance"
  - "com.example.label-with-empty-value"
```
容器添加元数据，和Dockerfile的LABEL指令意义相同
-   pid
```
pid: "host"
```
此处将PID模式设置为主机PID模式，跟主机系统共享进程命名空间。
-   networks
```
services:
  some-service:
    networks:
     - some-network
     - other-network
networks:
  some-network:
    driver: bridge
```
加入指定网络
-   volumes
```
volumes:
  // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
  - /var/lib/mysql

  // 使用绝对路径挂载数据卷
  - /opt/data:/var/lib/mysql

  // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
  - ./cache:/tmp/cache

  // 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
  - ~/configs:/etc/configs/:ro

  // 已经存在的命名的数据卷。
  - datavolume:/var/lib/mysql
```
挂载一个目录或者一个已存在的数据卷容器
-   volumes_from
```
volumes_from:
  - service_name
  - service_name:ro
  - container:container_name
  - container:container_name:rw
```
从其它容器或者服务挂载数据卷，可选的参数是 :ro或者 :rw，前者表示容器只读，后者表示容器对数据卷是可读可写的。默认情况下是可读可写的。


