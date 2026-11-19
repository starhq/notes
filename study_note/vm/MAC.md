# MacOS虚拟机OrbStack

## 初始化ubuntu

```yml
#cloud-config

apt:
  primary:
    - arches: [default]
      uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
  security:
    - arches: [default]
      uri: https://mirrors.tuna.tsinghua.edu.cn/ubuntu/
      
package_update: true
packages: [docker.io, docker-compose-v2]

write_files:
  - path: /etc/docker/daemon.json
    content: |
      {
        "registry-mirrors": [
          "https://docker.xuanyuan.me",
          "https://docker.1ms.run",
          "https://docker.m.daocloud.io"
        ]
      }

users:
  - default
  - name: wangjian 
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL

runcmd:
  - systemctl enable --now docker
  - usermod -aG docker ubuntu
  - usermod -aG docker wangjian

```

## 初始化alpine

```yml
#cloud-config

bootcmd:
  - printf '%s\n' 'http://mirrors.ustc.edu.cn/alpine/v3.23/main' 'http://mirrors.ustc.edu.cn/alpine/v3.23/community' > /etc/apk/repositories

package_update: true
packages:
  - docker
  - docker-cli-compose

write_files:
  - path: /etc/docker/daemon.json
    content: |
      {
        "registry-mirrors": [
          "https://docker.xuanyuan.me",
          "https://docker.1ms.run",
          "https://docker.m.daocloud.io"
        ]
      }

users:
  - default
  - name: wangjian
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL

runcmd:
  - rc-update add docker default
  - rc-service docker start
  - addgroup wangjian docker
```