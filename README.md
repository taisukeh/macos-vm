# macos-vm

## 資料

### Virtualization framework を利用したアプリ

- https://mac.getutm.app/

### WWDC

#### Create macOS or Linux virtual machines
https://developer.apple.com/videos/play/wwdc2022/10002/

#### Create seamless experiences with Virtualization

https://developer.apple.com/videos/play/wwdc2023/10007/

- Virtualization framework
- resizable display
- Disk block device
- I/O performance improvements
- Saving and restoring virtual machines
- Paravirtualized GPU security isoration
- Mac keyboard
- Network block device
- Rosetta2 caching
- USB 3.0 mass storage device
- NVMe controller device

- user experience
  - New features
  - Resizable display
    - VM作成時にSwiftコードでリサイズ可能化を指定する
  - Save a virtual machine
    - VMの状態を前に巻き戻せる
      - saveMachineState API
      - ディスクイメージなどの外部リソースは個別にコピーする必要がある
    - 保存ファイルは Hardware encrypted
      - 他のユーザー、Macでは復元できない
    - 保存ファイルはバージョン管理されている
  - New ways to build VM
    - Network block device
      - ストレージをネットワーク経由で仮想マシンに接続可能
      - 
    - NVMeのサポート
      - virtio ブロックデバイスの代替
      - Linuxマシン専用
    - Mac keyboard
      - 仮想マシンに直接接続
    - Rosetta2
      - VM Linux上でRosetta2がより高速に動作する
- configuration options
- Rosetta2


### VMについて

- ハイパーバイザーとは
  - https://www.redhat.com/ja/topics/virtualization/what-is-a-hypervisor
- KVMとは
  - https://www.redhat.com/ja/topics/virtualization/what-is-KVM
