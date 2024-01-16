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

### snapshot 

- qmue snapshot
  - https://kashyapc.fedorapeople.org/virt/lc-2012/snapshots-handout.html
  - https://github.com/Metamogul/UTM-Snapshot-Manager

- Parallels: macOS virtual machines on Mac computers with Apple silicon 
  - https://kb.parallels.com/128867
- VMWare Fusion
  https://www.vmware.com/jp/products/fusion.html
  Mac は Intel のみサポート
- CircleCI は　VMWare を利用している。無料枠では　Intel のみ利用可能だった。



1. qemu-nbd で NBD サーバー `--snapshot` オプションで起動
2. Virtualization Framework で NBD デバイスをマウント
```
qemu-nbd --snapshot -x myDisk Disk.img
```

raw img ではなく qcow のイメージも利用できる。

lsof
```
% sudo lsof -p 66181
COMMAND    PID   USER   FD   TYPE             DEVICE    SIZE/OFF      NODE NAME
qemu-nbd 66181 tahori  cwd    DIR               1,18         640         2 /
qemu-nbd 66181 tahori  txt    REG               1,18     2128560 152035546 /opt/homebrew/Cellar/qemu/8.2.0/bin/qemu-nbd
qemu-nbd 66181 tahori  txt    REG               1,18       71264 150874436 /opt/homebrew/Cellar/glib/2.78.3/lib/libgmodule-2.0.0.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      162464 150182576 /opt/homebrew/Cellar/gettext/0.22.4/lib/libintl.8.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      370080 150874437 /opt/homebrew/Cellar/glib/2.78.3/lib/libgobject-2.0.0.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      108160 117497187 /opt/homebrew/Cellar/libtasn1/4.19.0/lib/libtasn1.6.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      426800 153772981 /opt/homebrew/Cellar/libssh/0.10.6/lib/libssh.4.9.6.dylib
qemu-nbd 66181 tahori  txt    REG               1,18     1852528 148033700 /opt/homebrew/Cellar/gnutls/3.8.2/lib/libgnutls.30.dylib
qemu-nbd 66181 tahori  txt    REG               1,18     1824416 150874434 /opt/homebrew/Cellar/glib/2.78.3/lib/libgio-2.0.0.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      606368 150886749 /opt/homebrew/Cellar/pixman/0.42.2/lib/libpixman-1.0.42.2.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      258960 146891183 /opt/homebrew/Cellar/libidn2/2.3.4_1/lib/libidn2.0.dylib
qemu-nbd 66181 tahori  txt    REG               1,18     1199008 150874435 /opt/homebrew/Cellar/glib/2.78.3/lib/libglib-2.0.0.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      652704 150203206 /opt/homebrew/Cellar/zstd/1.5.5/lib/libzstd.1.5.5.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      300480 146863701 /opt/homebrew/Cellar/nettle/3.9.1/lib/libhogweed.6.8.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      333104 146863702 /opt/homebrew/Cellar/nettle/3.9.1/lib/libnettle.8.8.dylib
qemu-nbd 66181 tahori  txt    REG               1,18     1470720 148027077 /opt/homebrew/Cellar/p11-kit/0.25.3/lib/libp11-kit.0.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      452816 146863470 /opt/homebrew/Cellar/gmp/6.3.0/lib/libgmp.10.dylib
qemu-nbd 66181 tahori  txt    REG               1,18      544960 146870621 /opt/homebrew/Cellar/pcre2/10.42/lib/libpcre2-8.0.dylib
qemu-nbd 66181 tahori  txt    REG               1,18     1799040 117498047 /opt/homebrew/Cellar/libunistring/1.1/lib/libunistring.5.dylib
qemu-nbd 66181 tahori  txt    REG               1,18     4199552 150196465 /opt/homebrew/Cellar/openssl@3/3.2.0_1/lib/libcrypto.3.dylib
qemu-nbd 66181 tahori    0u   CHR               16,1   0t3510277       821 /dev/ttys001
qemu-nbd 66181 tahori    1u   CHR               16,1   0t3510277       821 /dev/ttys001
qemu-nbd 66181 tahori    2u   CHR               16,1   0t3510277       821 /dev/ttys001
qemu-nbd 66181 tahori    3u  unix 0x2d742bb7cc255ed7         0t0           ->0x2d742bb7cc250437
qemu-nbd 66181 tahori    4u  IPv4 0x2d742bae31ad7e57         0t0       TCP *:10809 (LISTEN)
qemu-nbd 66181 tahori    5   PIPE 0x5a1ed45ce5616227       16384           ->0x8ad575f3753290d1
qemu-nbd 66181 tahori    6   PIPE 0x8ad575f3753290d1       16384           ->0x5a1ed45ce5616227
qemu-nbd 66181 tahori    7   PIPE 0x5f24b88127ccc36b       16384           ->0x9fa28ee3c7fda40c
qemu-nbd 66181 tahori    8   PIPE 0x9fa28ee3c7fda40c       16384           ->0x5f24b88127ccc36b
qemu-nbd 66181 tahori    9   PIPE 0xf5a1c4bde299c888       16384           ->0xb2a5217fe8c82c7
qemu-nbd 66181 tahori   10   PIPE  0xb2a5217fe8c82c7       16384           ->0xf5a1c4bde299c888
qemu-nbd 66181 tahori   11   PIPE 0xbc5034f6f38a1122       16384           ->0xe50f049a809c7142
qemu-nbd 66181 tahori   12   PIPE 0xe50f049a809c7142       16384           ->0xbc5034f6f38a1122
qemu-nbd 66181 tahori   13r   REG               1,18 34359738368 151860252 /Users/tahori/VM.bundle/Disk.img
qemu-nbd 66181 tahori   14u   REG               1,18   853409792 155143496 /private/var/folders/7r/kvbmpqmx2q9d0ys432__v3_h0000gn/T/vl.GGQOH2
qemu-nbd 66181 tahori   15u  IPv4 0x2d742bae3049a297         0t0       TCP localhost:10809->localhost:64531 (ESTABLISHED)
```
