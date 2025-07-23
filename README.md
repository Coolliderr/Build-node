# 🚀 BSC 全节点部署教程（Hetzner 专用）

## 📌 1. 重装系统

1✅、在 Hetzner 控制台或 Rescue 模式执行：
```bash
installimage -a
```

2✅、示例自动配置文件 /installimage.auto/install.conf
```bash
DRIVE1 /dev/nvme0n1
SWRAID 0

HOSTNAME bsc-node
USE_KERNEL_MODE_SETTING yes

PART /boot ext4 1G
PART / ext4 all

IMAGE /root/.oldroot/nfs/images/Ubuntu-2204-jammy-amd64-base.tar.gz
AUTOSTART 1
```

3✅、执行安装：
```bash
installimage -a -c /installimage.auto/install.conf
reboot
```

4✅、登录后验证
```bash
lsb_release -a
lsblk -f
df -h
```

## 📌 2. 挂载数据盘（7T）
  
1✅、查看 UUID：
```bash
blkid /dev/nvme2n1
```

2✅、编辑 /etc/fstab
```bash
nano /etc/fstab
```

3✅、添加
```bash
UUID=<你的UUID> /data ext4 defaults 0 2
```

4✅、挂载并验证
```bash
mount -a && df -h
```

## 📌 3. 安装依赖

```bash
apt update && apt install -y git curl wget unzip build-essential jq
```

## 📌 4. 安装 Go

```bash
cd /data/bsc
wget https://go.dev/dl/go1.24.0.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.24.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
go version
```

## 📌 5. 下载并编译 BSC

```bash
cd /data/bsc
git clone https://github.com/bnb-chain/bsc.git
cd bsc
make geth
```

## 📌 6. 初始化数据目录

1✅、下载最新主网配置
```bash
wget $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest | grep browser_ | grep mainnet | cut -d\" -f4)
unzip *.zip
```

2✅、初始化
```bash
./build/bin/geth --datadir ./node init genesis.json
```

## 📌 7. 启动节点（同步区块）

```bash
./build/bin/geth \
  --config ./config.toml \
  --datadir ./node \
  --cache 16000 \
  --rpc.allow-unprotected-txs
```

## 📌 8. 查看同步状态

```bash
curl -s http://127.0.0.1:8545 \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
```

```bash
{"jsonrpc":"2.0","id":1,"result":false}
```

## 📌 9. 使用 PM2 守护进程

1✅、创建启动脚本 /data/bsc/start-bsc.sh
```bash
nano /data/bsc/start-bsc.sh
```

2✅、内容
```bash
#!/bin/bash
cd /data/bsc
./build/bin/geth \
  --config ./config.toml \
  --datadir ./node \
  --cache 16000 \
  --rpc.allow-unprotected-txs
```

3✅、赋权并启动
```bash
chmod +x /data/bsc/start-bsc.sh
pm2 start /data/bsc/start-bsc.sh --name bsc-node
pm2 save
pm2 ls
pm2 logs bsc-node
```

## 📌 ✅ 检查同步进度

```bash
# 查询当前区块高度
curl -s -X POST http://127.0.0.1:8545 \
-H "Content-Type: application/json" \
--data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
```
