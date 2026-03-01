--- name: network-cli
description: Natural language control for Huawei/H3C/Ruijie routers and switches. Use when user wants to configure network devices via natural language commands like "create VLAN 10", "configure interface GigabitEthernet 0/0/1 with IP 192.168.1.1", "show running configuration", "backup switch config", "add port to VLAN". Translates natural language to CLI commands and executes them via SSH.
---

# 自然语言网络设备控制 (Network CLI)

## 支持的设备
- **华为 (Huawei)** - VRP 系统
- **H3C (新华三)** - Comware 系统
- **锐捷 (Ruijie)** - RGOS 系统
- **TP-Link Omada** - 企业级设备

## 核心功能

### 1. 自然语言 → CLI 命令转换

```
用户说: "创建VLAN 10"
生成: vlan 10

用户说: "给GigabitEthernet 0/0/1配置IP地址192.168.1.1/24"
生成: interface GigabitEthernet 0/0/1
      ip address 192.168.1.1 255.255.255.0

用户说: "查看当前配置"
生成: display current-configuration

用户说: "保存配置"
生成: save

用户说: "把端口1加入VLAN 10"
生成: interface GigabitEthernet 0/0/1
      port link-type access
      port default vlan 10

用户说: "配置Trunk端口允许VLAN 10,20,30通过"
生成: interface GigabitEthernet 0/0/1
      port link-type trunk
      port trunk permit vlan 10 20 30
```

### 2. 命令自动适配品牌

| 功能 | 华为命令 | H3C命令 | 锐捷命令 |
|------|---------|---------|---------|
| **系统视图** | system-view | system-view | configure terminal |
| **退出配置** | quit / return | quit / return | end |
| **设备改名** | sysname SW1 | sysname SW1 | hostname SW1 |
| **创建VLAN 10** | vlan 10 | vlan 10 | vlan 10 |
| **批量创建VLAN** | vlan batch 10 20 30 | vlan 10 to 20 / vlan batch 10 20 | vlan 10,20,30 |
| **进入接口** | interface GigabitEthernet 0/0/1 | interface GigabitEthernet 1/0/1 | interface GigabitEthernet 0/1 |
| **Access端口** | port link-type access<br>port default vlan 10 | port link-type access<br>port access vlan 10 | switchport mode access<br>switchport access vlan 10 |
| **Trunk端口** | port link-type trunk<br>port trunk allow-pass vlan 10 20 | port link-type trunk<br>port trunk permit vlan 10 20 | switchport mode trunk<br>switchport trunk allowed vlan add 10,20 |
| **VLANIF三层** | interface Vlanif 10 | interface Vlan-interface 10 | interface vlan 10 |
| **查看配置** | display current-configuration | display current-configuration | show running-config |
| **保存配置** | save | save | wr 或 write |
| **查看VLAN** | display vlan | display vlan | show vlan brief |
| **ping命令** | ping 192.168.1.1 | ping 192.168.1.1 | ping 192.168.1.1 |
| **开启SSH** | stelnet server enable<br>ssh user admin | ssh server enable<br>local-user admin | ip ssh server enable |

### 3. SSH自动连接与执行

```python
import paramiko

def connect_router(ip, username, password, device_type="huawei"):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(ip, username=username, password=password)
    return ssh

def execute_cmd(ssh, commands, device_type="huawei"):
    channel = ssh.invoke_shell()
    time.sleep(0.5)
    
    # 进入系统视图
    if device_type == "huawei":
        channel.send("system-view\n")
    elif device_type == "h3c":
        channel.send("system-view\n")
    elif device_type == "ruijie":
        channel.send("configure terminal\n")
    
    # 逐条执行命令
    for cmd in commands:
        channel.send(cmd + "\n")
        time.sleep(0.5)
    
    # 接收输出
    output = channel.recv(4096).decode()
    return output
```

## 使用示例

### 批量配置VLAN
```
用户: "在192.168.1.100上创建VLAN 10、20、30，每个VLAN配置网关"

自动执行:
ssh 192.168.1.100
system-view
vlan batch 10 20 30
interface Vlanif 10
 ip address 192.168.10.1 24
interface Vlanif 20
 ip address 192.168.20.1 24
interface Vlanif 30
 ip address 192.168.30.1 24
save
```

### 端口批量配置
```
用户: "把端口1-8都配成access模式，分别加入VLAN 10-17"

自动执行:
for i in range(1, 9):
    interface GigabitEthernet 0/0/{i}
    port link-type access
    port default vlan {9+i}
```

### 备份配置
```
用户: "备份192.168.1.100的配置到本地"

自动执行:
ssh 192.168.1.100
display current-configuration > config_backup.txt
tftp 192.168.1.200 put config_backup.txt startup.cfg
```

## 安全机制

1. **命令预览** - 执行前显示生成的CLI命令
2. **配置备份** - 自动备份当前配置
3. **回滚准备** - 准备好回滚命令
4. **确认提示** - 高危操作（删除、reset）需确认
5. **日志记录** - 记录所有操作

## 协议支持

- **SSH** - 主要管理方式
- **Telnet** - 备选（不推荐）
- **SNMP** - 监控和读取配置
- **TFTP/FTP** - 配置备份和恢复

## 自然语言命令映射 (华为VRP)

### VLAN配置完整命令表

| 自然语言 | CLI命令序列 | 说明 |
|---------|------------|------|
| "进入系统视图" | `system-view` | 从用户视图进入系统视图 |
| "修改设备名称为SW1" | `sysname SW1` | 可选，用于标识设备 |
| "创建VLAN 10" | `vlan 10`<br>`name VLAN_Test`<br>`quit` | 创建并命名VLAN |
| "批量创建VLAN 10,20,30" | `vlan batch 10 20 30` | 批量创建多个VLAN |
| "批量创建VLAN 10到20" | `vlan batch 10 to 20` | 创建连续VLAN范围 (10-20) |
| "端口1加入VLAN 10 (Access)" | `interface GigabitEthernet 0/0/1`<br>`port link-type access`<br>`port default vlan 10`<br>`quit` | Access端口配置 |
| "配置Trunk允许VLAN 10,20" | `interface GigabitEthernet 0/0/24`<br>`port link-type trunk`<br>`port trunk allow-pass vlan 10 20`<br>`quit` | Trunk端口配置 |
| "Trunk允许所有VLAN" | `port trunk allow-pass vlan all` | 允许所有VLAN通过 |
| "配置Hybrid端口" | `interface GigabitEthernet 0/0/5`<br>`port link-type hybrid`<br>`port hybrid pvid vlan 20`<br>`port hybrid untagged vlan 20`<br>`quit` | 华为特有Hybrid模式 |
| "给VLAN 10配网关" | `interface Vlanif 10`<br>`ip address 192.168.10.1 255.255.255.0`<br>`quit` | 三层VLAN接口配置 |
| "配置静态路由" | `ip route-static 0.0.0.0 0.0.0.0 192.168.1.1` | 默认路由配置 |
| "查看当前配置" | `display current-configuration` | 显示完整配置 |
| "查看VLAN信息" | `display vlan` | 显示所有VLAN |
| "查看VLAN 10详情" | `display vlan 10` | 显示特定VLAN信息 |
| "查看接口信息" | `display interface GigabitEthernet 0/0/1` | 显示接口状态 |
| "保存配置" | `save` | 保存到启动配置文件 |
| "开启SSH服务" | `stelnet server enable`<br>`ssh user admin`<br>`ssh user admin authentication-type password`<br>`ssh user admin service-type stelnet` | SSH远程管理配置 |

### 端口模式对比

| 模式 | 用途 | 数据帧处理 | 典型场景 |
|-----|------|-----------|---------|
| **Access** | 连接终端设备 | 入口加Tag，出口剥Tag | PC、打印机、IP电话 |
| **Trunk** | 连接交换机/路由器 | 保持VLAN Tag，允许多个VLAN | 交换机之间互联 |
| **Hybrid** | 华为特有灵活模式 | 可控制出入口Tag行为 | 特殊场景，如IP电话+PC |

### Python SSH自动化 (Netmiko vs Paramiko)

**Paramiko** - 底层SSH控制
```python
import paramiko
import time

def huawei_config_paramiko(ip, username, password, commands):
    """使用Paramiko配置华为交换机"""
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(ip, username=username, password=password, look_for_keys=False)
    
    channel = ssh.invoke_shell()
    time.sleep(0.5)
    
    # 进入系统视图
    channel.send("system-view\n")
    time.sleep(0.3)
    
    # 执行命令列表
    for cmd in commands:
        channel.send(cmd + "\n")
        time.sleep(0.3)
    
    # 保存配置
    channel.send("save\n")
    time.sleep(0.2)
    channel.send("Y\n")  # 确认保存
    time.sleep(0.2)
    channel.send("Y\n")
    time.sleep(0.5)
    
    output = channel.recv(65535).decode('utf-8', errors='ignore')
    ssh.close()
    return output

# 使用示例
commands = [
    "vlan batch 10 20 30",
    "interface Vlanif 10",
    "ip address 192.168.10.1 24",
    "quit",
    "interface GigabitEthernet 0/0/1",
    "port link-type access",
    "port default vlan 10"
]
result = huawei_config_paramiko('192.168.1.100', 'admin', 'Huawei@123', commands)
print(result)
```

**Netmiko** - 高层网络设备抽象（推荐）
```python
from netmiko import ConnectHandler

def huawei_config_netmiko(ip, username, password, commands):
    """使用Netmiko配置华为交换机 - 更简洁"""
    device = {
        'device_type': 'huawei',
        'ip': ip,
        'username': username,
        'password': password,
        'port': 22,
    }
    
    conn = ConnectHandler(**device)
    conn.enable()  # 进入特权模式
    
    # 自动处理prompt和命令发送
    output = conn.send_config_set(commands)
    
    # 保存配置
    save_output = conn.send_command("save", expect_string=r"\[Y/N\]")
    save_output += conn.send_command("Y", expect_string=r"\[Y/N\]")
    save_output += conn.send_command("Y")
    
    conn.disconnect()
    return output + "\n" + save_output

# 使用示例 - 批量配置VLAN
device_info = {
    'device_type': 'huawei',
    'ip': '192.168.1.100',
    'username': 'admin',
    'password': 'Huawei@123',
}

commands = [
    'vlan batch 10 20 30',
    'interface Vlanif 10',
    'ip address 192.168.10.1 255.255.255.0',
    'description Office_VLAN',
    'quit',
]

# 批量管理多台设备
devices = [
    {'ip': '192.168.1.100', 'name': 'Switch-Core'},
    {'ip': '192.168.1.101', 'name': 'Switch-Access1'},
]

for dev in devices:
    device_info['ip'] = dev['ip']
    conn = ConnectHandler(**device_info)
    print(f"Configuring {dev['name']}...")
    output = conn.send_config_set(commands)
    conn.save_config()  # Netmiko内置保存方法
    conn.disconnect()
```

**设备类型支持**
| 品牌 | Netmiko device_type | 备注 |
