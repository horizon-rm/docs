# docker 安装

## 1. docker 软件安装

```bash
# 更新软件包列表
sudo apt-get update

# 安装基础依赖
sudo apt-get -y install ca-certificates curl

# 创建密钥目录
sudo install -m 0755 -d /etc/apt/keyrings

# 下载 Docker 官方 GPG 密钥（阿里云镜像）
sudo curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# 添加 Docker APT 源（阿里云镜像）
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://mirrors.aliyun.com/docker-ce/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list

# 再次更新软件包列表
sudo apt-get update

# 安装 Docker
sudo apt-get -y install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```


## 2. docker 代理配置

### 方法1 使用镜像加速器

```bash
# 已成功
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": [
    "https://docker.m.daocloud.io"
  ]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker

# 检验是否能够成功拉取
sudo docker run hello-world
```

其他代理配置： 
```bash
cat /etc/docker/daemon.json 
# 其他的镜像，大多不能用
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://ueo0uggy.mirror.aliyuncs.com",
    "https://docker.m.daocloud.io",
    "https://cf-workers-docker-io-apl.pages.dev",
  ]
}

# 更改镜像地址后可以测试一下
 curl -v  <镜像地址>
```

### 方法2 使用http代理

```bash
sudo vim /etc/docker/daemon.json
{
  "proxies": 
  {
    "http-proxy": "http://127.0.0.1:7890",
    "https-proxy": "http://127.0.0.1:7890"
  }
}
sudo systemctl restart docker
sudo docker info | grep -A 1 ' HTTP Proxy'
```

### 方法3 docker.service 代理

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d &&
sudo tee /etc/systemd/system/docker.service.d/proxy.conf <<EOF
[Service]
Environment="HTTP_PROXY=http://127.0.0.1:7890/"
Environment="HTTPS_PROXY=http://127.0.0.1:7890/"
EOF
sudo systemctl daemon-reload &&
sudo systemctl restart docker
```

请自行把 `http://127.0.0.1:7890` 改为合适的代理地址。
