## awesome command

### Docker
- 查询镜像带指定关键字的`ID`（如`keyword`）
  - `docker images|grep keyword|awk -F" " '{ print $3 }' -| xargs`
  - `docker images --filter=reference="keyword:*" --format "{{ .ID }}| xargs"`
- 查询`TAG`为`<none>`的镜像
  - `docker images|awk -F" " '{ if($2=="<none>") print $3 }' -|xargs`
- 查询前十个镜像
  - `docker images|awk -F" " '{print $3 }' -|head -n 10`
  - `docker images --format "{{.ID}}"|head -n 10`

### Linux
- 清除缓存
  - page_cache：`sync; echo 1 > /proc/sys/vm/drop_caches`
  - dentry/inode：`sync; echo 2 > /proc/sys/vm/drop_caches`
  - page_cache/dentry/inode：`sync; echo 3 > /proc/sys/vm/drop_caches`
- 追加组群：`usermod -G [group1],[group2]... -a [username]`
- 查看加载内核信息： `dmesg`

### Kubernetes
- 转发`Pod=application,port=8080`上的流量到`localhost:8081`上：`kubectl port-forward application --address localhost 8081:8080`

### Git
- 大文件支持：`git lfs install`
- 对象查询：`git cat-file [sha1] -t/-s/-p`
- `windows`系统取消`\r\n`换行：`git config core.autocrlf false`

### Vim(~/.vimrc)
- 文本格式转为`Unix`：`:set fileformat=unix`
- 显示行号：`:set number`

### JAVA
- 开启远程调试： `java -Xdebug -Xrunjdwp:transport=dt_socket,server=y,address=[port],suspend=n -jar [jar-name].jar`
- 连接远程调试(dll)： `java -agentlib:transport=dt_socket,server=y,suspend=y,address=[ip:port] -jar [jar-name].jar`

### NFS
- 解挂： `unmount [target-path]`
#### NFS
- 挂载： `mount -t nfs [source-ip]:[source-path] [target-path]`
#### CIFS/Samba
- 挂载： `mount -t cifs -o username=[username],password=[password] //[ip:port]/[source-path] [target-path]` 

### Firewall
- 所有出入站规则（cidr 显示）： `iptables -L -n`
- 清空所有规则： `iptables -F`
- 规则添加： `iptables [action[-I/-A/-P/-D/-R/-E]] [INPUT/OUTPUT] -s [source-ip] -p [protocol[tcp/icmp]] --dport [target-port] -j [ACCEPT/DROP]`
- 禁止转发： `iptables -P FORWARD DROP`

### MOD
- 查看配置： `mobprobe -c`
- 加载： `modprobe ipip`
- 卸载已加载的模块： `modprobe -r ipip`
- 具体信息： `modinfo ipip`

### TUNNEL(CLOUD-SDN-VPC)
```
# 打通不同网段隧道连接
# IP in IP/最小封装/通用路由封装
mobprobe [ipip/sit/gre]
ip tunnel add [dev-name] mode [ipip/gre/sit] remote [source-ip] local [target-outer-ip] ttl 64  
ip link set [dev-name] up  
ip addr add [source-inter-ip] peer [target-inter-ip] dev [dev-name]
ip route add [target-inter-cidr] dev [dev-name]
(optional) iptables -F
```