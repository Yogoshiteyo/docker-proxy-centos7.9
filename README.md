# CentOS 7.9 安装 Docker 并使用代理加速访问 Docker 仓库拉取镜像

## 背景说明
由于国内网络限制，直接访问Docker官方仓库速度较慢或不稳定。本方案通过socks5代理服务实现：
1. CentOS 7系统基础环境配置
2. 全局代理搭建
3. Docker服务代理配置
4. 一键部署脚本在最后
<font color=red>PS：因屏蔽词的原因，请将"贰"改为"2"</font>
---

## 前置条件
- 操作系统：CentOS 7.9-2009（最小化安装）
- 网络要求：虚拟机需具备访问外网能力
- 权限要求：root用户或sudo权限
- 梯子要求：在v贰ray中能使用的梯子都可以
- 其他要求：一台Windows电脑，并安装v贰ray客户端软件。关于v贰ray的使用请参照：
https://www.cnblogs.com/yogochann/p/18714813
---

## 部署流程

### 1. 系统环境准备
#### 1.1 更换阿里云镜像源
# 备份原有仓库文件
```bash
mkdir -p /etc/yum.repos.d/backup
mv /etc/yum.repos.d/*.repo /etc/yum.repos.d/backup/ 2>/dev/null
```
# 下载阿里云仓库文件
```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
```
# 添加Docker仓库
```bash
curl -o /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
# 清理缓存
```bash
yum clean all && yum makecache
```
#### 1.2 安装基础组件
```bash
yum install -y vim gzip net-tools unzip firewalld docker-ce wget
systemctl enable --now docker firewalld
```
## 2. v贰ray服务部署
### 2.1 获取安装包
```bash
wget -P /tmp https://github.com/v2fly/v贰ray-core/releases/download/v4.31.0/v贰ray-linux-64.zip
```
### 2.2 安装配置
- 解压文件
```bash
mkdir -p /usr/local/v贰ray
unzip /tmp/v贰ray-linux-64.zip -d /usr/local/v贰ray
```
- 替换配置文件（需提前准备config.json）
```bash
cp flyingbird/config.json /usr/local/v贰ray/
```
### 2.3 创建服务单元
- 使用tee创建v贰ray.service文件
```bash
tee /etc/systemd/system/v贰ray.service <<'EOF'
[Unit]
Description=v贰ray Service
After=network.target

[Service]
ExecStart=/usr/local/v贰ray/v贰ray run -config /usr/local/v贰ray/config.json
Restart=always

[Install]
WantedBy=multi-user.target
EOF
```
- 重载服务
```bash
systemctl daemon-reload
systemctl enable --now v贰ray
```
### 2.4 防火墙配置
```bash
firewall-cmd --permanent --add-port={10808,10809}/tcp
firewall-cmd --reload
```
## 3. Docker代理配置
### 3.1 创建代理配置文件
```bash
mkdir -p /etc/systemd/system/docker.service.d

tee /etc/systemd/system/docker.service.d/http-proxy.conf <<'EOF'
[Service]
Environment="HTTP_PROXY=socks5://127.0.0.1:10808"
Environment="HTTPS_PROXY=socks5://127.0.0.1:10808"
EOF
```
### 3.2 应用配置
```bash
systemctl daemon-reload
systemctl restart docker
```  
### 3.3 拉取镜像
![](https://img2024.cnblogs.com/blog/3158231/202502/3158231-20250217113235654-426199887.png)

## 4. 一键脚本
### 4.1 前置工作
#### 4.1.1 准备好json文件。
- 可以在Windows版的v贰ray中获取。选中所需节点，点击鼠标右键"导出所选服务器完整配置"。
![](https://img2024.cnblogs.com/blog/3158231/202502/3158231-20250217152423668-1893413302.png)
- 将配置文件保存到flyingbird文件夹中，命名为config。（后缀名会自动识别为json）
![](https://img2024.cnblogs.com/blog/3158231/202502/3158231-20250217160215550-2025233684.png)
- 将flyingbird文件夹上传至centos7主机
![](https://img2024.cnblogs.com/blog/3158231/202502/3158231-20250217163633206-1531941476.png)

#### 4.1.2 准备好v贰ray的安装包。
https://github.com/v2fly/v贰ray-core/releases/download/v5.24.0/v贰ray-linux-64.zip
### 4.2 运行脚本
```bash
curl -o deploy.sh https://files-cdn.cnblogs.com/files/blogs/827077/deploy.sh?t=1739773875 && chmod +x deploy.sh && ./deploy.sh
```
### 4.3 安装成功
![](https://img2024.cnblogs.com/blog/3158231/202502/3158231-20250217140228372-662145035.png)
