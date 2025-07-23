🚀 BSC 全节点部署教程（Hetzner 专用）
📌 1. 重装系统
在 Hetzner 控制台或 Rescue 模式执行：

bash
复制
编辑
installimage -a
示例自动配置文件 /installimage.auto/install.conf
bash
复制
编辑
DRIVE1 /dev/nvme0n1
SWRAID 0

HOSTNAME bsc-node
USE_KERNEL_MODE_SETTING yes

PART /boot ext4 1G
PART / ext4 all

IMAGE /root/.oldroot/nfs/images/Ubuntu-2204-jammy-amd64-base.tar.gz
AUTOSTART 1
执行安装：

bash
复制
编辑
installimage -a -c /installimage.auto/install.conf
reboot
登录后验证：

bash
复制
编辑
lsb_release -a
lsblk -f
df -h
📌 2. 挂载数据盘（7T）
查看 UUID：

bash
复制
编辑
blkid /dev/nvme2n1
编辑 /etc/fstab：

bash
复制
编辑
nano /etc/fstab
添加：

ini
复制
编辑
UUID=<你的UUID> /data ext4 defaults 0 2
挂载并验证：

bash
复制
编辑
mount -a && df -h
📌 3. 安装依赖
bash
复制
编辑
apt update && apt install -y git curl wget unzip build-essential jq
📌 4. 安装 Go
bash
复制
编辑
cd /data/bsc
wget https://go.dev/dl/go1.24.0.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.24.0.linux-amd64.tar.gz
export PATH=$PATH:/usr/local/go/bin
echo 'export PATH=/usr/local/go/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
go version
📌 5. 下载并编译 BSC
bash
复制
编辑
cd /data/bsc
git clone https://github.com/bnb-chain/bsc.git
cd bsc
make geth
编译成功后，geth 在：

bash
复制
编辑
./build/bin/geth
📌 6. 初始化数据目录
下载最新主网配置：

bash
复制
编辑
wget $(curl -s https://api.github.com/repos/bnb-chain/bsc/releases/latest | grep browser_ | grep mainnet | cut -d\" -f4)
unzip *.zip
初始化：

bash
复制
编辑
./build/bin/geth --datadir ./node init genesis.json
📌 7. 启动节点（同步区块）
bash
复制
编辑
./build/bin/geth \
  --config ./config.toml \
  --datadir ./node \
  --cache 16000 \
  --rpc.allow-unprotected-txs
📌 8. 查看同步状态
bash
复制
编辑
curl -s http://127.0.0.1:8545 \
  -X POST \
  -H "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0","method":"eth_syncing","params":[],"id":1}'
false → 同步完成。

对象 → 查看 currentBlock、highestBlock。

📌 9. 使用 PM2 守护进程
创建启动脚本 /data/bsc/start-bsc.sh：

bash
复制
编辑
nano /data/bsc/start-bsc.sh
内容：

bash
复制
编辑
#!/bin/bash
cd /data/bsc
./build/bin/geth \
  --config ./config.toml \
  --datadir ./node \
  --cache 16000 \
  --rpc.allow-unprotected-txs
赋权并启动：

bash
复制
编辑
chmod +x /data/bsc/start-bsc.sh
pm2 start /data/bsc/start-bsc.sh --name bsc-node
pm2 save
pm2 ls
pm2 logs bsc-node
✅ 检查同步进度
bash
复制
编辑
# 查询当前区块高度
curl -s -X POST http://127.0.0.1:8545 \
-H "Content-Type: application/json" \
--data '{"jsonrpc":"2.0","method":"eth_blockNumber","params":[],"id":1}'
⚠️ 注意
当前模式 = 快照节点（不保存历史状态），硬盘占用约 40-60GB。

如果需要 完整历史数据（Archive Node），磁盘需求 >6TB。

🔗 官方资源
BSC GitHub

BSC 文档

