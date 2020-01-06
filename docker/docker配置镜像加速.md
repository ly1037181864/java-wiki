###docker配置镜像加速

- 配置阿里云镜像加速
```text
cd /etc/docker/
vim daemon.json
```
daemon.json内容
```text
{
  "registry-mirrors": ["https://vantnh5v.mirror.aliyuncs.com"]
}
```
执行命令
```text
systemctl daemon-reload
systemctl restart docker
```