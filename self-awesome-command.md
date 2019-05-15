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
- 追加组群：`usermod -G -a [group1],[group2]... [username]`

### Kubernetes
- 转发`localhost:8081`上的流量到`Pod=application`的端口`8080`上：`kubectl port-forward application --address localhost 8081:8080`

### Git
- 大文件支持：`git lfs install`
- 对象查询：`git cat-file [sha1] -t/-s/-p`
- `windows`系统取消`\r\n`换行：`git config core.autocrlf false`

### Vim(~/.vimrc)
- 文本格式转为`Unix`：`:set fileformat=unix`
- 显示行号：`:set number`

### NFS
- 挂载：`mount -t nfs [source-ip]:[source-path] [target-path]`
- 解挂：`unmount [target-path]`