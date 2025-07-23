## Ubuntu设置静态IP



sudo apt install vim


cd /etc/netplan

sudo cp 01-network-manager-all.yaml  011-network-manager-all.yaml 

sudo vim 01-network-manager-all.yaml
sudo vi 01-network-manager-all.yaml

解决步骤：
🔒 1. 修复文件权限（防止安全隐患）
当前权限太开放（比如可能是644），需要改为仅所有者可读写（600）：

sudo chmod 600 /etc/netplan/01-network-manager-all.yaml
🛠 2. 修正静态IP配置格式错误
错误提示指出第9行：addresses: 192.168.25.22 格式错误。
正确格式：静态IP地址应该是一个列表（用中括号 [] 包裹），并且要同时包含子网掩码。

错误示例（当前配置）：
      addresses: 192.168.25.22  # 静态ip
正确示例（修改后）：
      addresses: [192.168.25.22/24]   # 静态IP必须带子网掩码（如24代表255.255.255.0）
注意：/24 是子网掩码的CIDR表示，请根据你的实际网络环境修改（常见值有16,24,25等）。

sudo rm -i /etc/netplan/011-network-manager-all.yaml

network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:   # 网卡名称
      dhcp4: no     # 关闭dhcp
      dhcp6: no
      addresses: [192.168.25.22/24]  # 带掩码的静态IP
      routes:
        - to: default
          via: 192.168.25.2           # 网关
      nameservers:
        addresses: [8.8.8.8, 114.114.114.114] #dns

sudo netplan generate
sudo netplan apply
