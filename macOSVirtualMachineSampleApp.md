macOSVirtualMachineSampleApp の内容解析
====


## InstallationTool

コマンドラインツール。

*引数を指定しない場合*

`VM.bundle` という名前のディレクトリを作成

`VZMacOSRestoreImage.fetchLatestSupported(completionHandler:)` を呼び出し `RestoreImage.ipsw` の URL を取得。

- 取得したURLは以下となっていた。
`https://updates.cdn-apple.com/2023FallFCS/fullrestores/052-15117/DC2EE605-ABF3-41AE-9652-D137A8AA5907/UniversalMac_14.2_23C64_Restore.ipsw"`


上記のURLからファイルをダウンロードしローカルに保存。

```
VZMacOSRestoreImage class func load(
    from fileURL: URL,
    completionHandler: @escaping (Result<VZMacOSRestoreImage, Error>) -> Void
)
```
を呼び出し ipsw ファイルをロードし、VZMacOSRestoreImageクラスのインスタンスを生成。


`VZMacOSRestoreImage var mostFeaturefulSupportedConfiguration: VZMacOSConfigurationRequirements? { get }`
で現在のホストとイメージで最大限サポートされる設定を取得。ここで取得する設定の型は `VZMacOSConfigurationRequirements`。



`VZVirtualMachineConfiguration` を生成。

- `VZMacOSConfigurationRequirements`
> An object that describes the parameter constraints required by a specific configuration of macOS.

- `VZVirtualMachineConfiguration`
> The environment attributes and list of devices to use during the configuration of macOS or Linux VMs.


#### `VZMacPlatformConfiguration` 作成
`VZMacPlatformConfiguration` を生成。
```
    private func createMacPlatformConfiguration(macOSConfiguration: VZMacOSConfigurationRequirements) -> VZMacPlatformConfiguration {
        let macPlatformConfiguration = VZMacPlatformConfiguration()

        guard let auxiliaryStorage = try? VZMacAuxiliaryStorage(creatingStorageAt: auxiliaryStorageURL,
                                                                    hardwareModel: macOSConfiguration.hardwareModel,
                                                                          options: []) else {
            fatalError("Failed to create auxiliary storage.")
        }
        macPlatformConfiguration.auxiliaryStorage = auxiliaryStorage
        macPlatformConfiguration.hardwareModel = macOSConfiguration.hardwareModel
        macPlatformConfiguration.machineIdentifier = VZMacMachineIdentifier()

        // Store the hardware model and machine identifier to disk so that you
        // can retrieve them for subsequent boots.
        try! macPlatformConfiguration.hardwareModel.dataRepresentation.write(to: hardwareModelURL)
        try! macPlatformConfiguration.machineIdentifier.dataRepresentation.write(to: machineIdentifierURL)

        return macPlatformConfiguration
    }
```

`VZMacAuxiliaryStorage` はゲストOSの boot loader で利用される 補助ストレージ（auxiliary storage）。
> An object that contains information the boot loader needs for booting macOS as a guest operating system.

> When moving or performing a backup of a VM, you must move or copy the file containing the auxiliary storage along with the main disk image.

VMのバックアップを行う際はディスクイメージとともにこの補助ストレージも必要になる。


`VZMacPlatformConfiguration` は起動するVMのプラットフォーム?の設定。

プロパティは３つ。
- `var auxiliaryStorage: VZMacAuxiliaryStorage?`
  - The Mac auxiliary storage.
- `var hardwareModel: VZMacHardwareModel`
  - The Mac hardware model.
- `var machineIdentifier: VZMacMachineIdentifier`
  - The Mac machine identifier.

hardwareModel は `VZMacAuxiliaryStorage` と一致させる必要がある。



#### VM 初期化

```
    private func setupVirtualMachine(macOSConfiguration: VZMacOSConfigurationRequirements) {
        let virtualMachineConfiguration = VZVirtualMachineConfiguration()

        virtualMachineConfiguration.platform = createMacPlatformConfiguration(macOSConfiguration: macOSConfiguration)
        virtualMachineConfiguration.cpuCount = MacOSVirtualMachineConfigurationHelper.computeCPUCount()
        if virtualMachineConfiguration.cpuCount < macOSConfiguration.minimumSupportedCPUCount {
            fatalError("CPUCount isn't supported by the macOS configuration.")
        }

        virtualMachineConfiguration.memorySize = MacOSVirtualMachineConfigurationHelper.computeMemorySize()
        if virtualMachineConfiguration.memorySize < macOSConfiguration.minimumSupportedMemorySize {
            fatalError("memorySize isn't supported by the macOS configuration.")
        }

        // Create a 128 GB disk image.
        createDiskImage()

        virtualMachineConfiguration.bootLoader = MacOSVirtualMachineConfigurationHelper.createBootLoader()
        virtualMachineConfiguration.graphicsDevices = [MacOSVirtualMachineConfigurationHelper.createGraphicsDeviceConfiguration()]
        virtualMachineConfiguration.storageDevices = [MacOSVirtualMachineConfigurationHelper.createBlockDeviceConfiguration()]
        virtualMachineConfiguration.networkDevices = [MacOSVirtualMachineConfigurationHelper.createNetworkDeviceConfiguration()]
        virtualMachineConfiguration.pointingDevices = [MacOSVirtualMachineConfigurationHelper.createPointingDeviceConfiguration()]
        virtualMachineConfiguration.keyboards = [MacOSVirtualMachineConfigurationHelper.createKeyboardConfiguration()]

        try! virtualMachineConfiguration.validate()

        if #available(macOS 14.0, *) {
            try! virtualMachineConfiguration.validateSaveRestoreSupport()
        }

        virtualMachine = VZVirtualMachine(configuration: virtualMachineConfiguration)
        virtualMachineResponder = MacOSVirtualMachineDelegate()
        virtualMachine.delegate = virtualMachineResponder
    }
```


`VZVirtualMachineConfiguration` について。

- VMの環境を設定するためのオブジェクト
- devices that the VM exposes to the guest operating system
  - ネットワークインタフェースやストレージなど
  - デバイスの詳細は https://developer.apple.com/documentation/virtualization の Devices セクションを参照

> Use a VZVirtualMachineConfiguration object to configure the environment for a macOS or Linux VM. This configuration object contains information about the VM environment, including the devices that the VM exposes to the guest operating system. For example, use the configuration object to specify the network interfaces and storage devices that the operating system may access. For more information on the devices that macOS and Linux guests can support, see the Devices section on the Virtualization framework page.


### `MacOSVirtualMachineConfigurationHelper`

CPUのコア数
```
    static func computeCPUCount() -> Int {
        # ホストOS上のプロセッサの数を取得
        let totalAvailableCPUs = ProcessInfo.processInfo.processorCount

        # ホストOS上のプロセッサ数 - 1にする
        var virtualCPUCount = totalAvailableCPUs <= 1 ? 1 : totalAvailableCPUs - 1
        
        # ゲストOSで利用できる最小値と最大値の処理
        virtualCPUCount = max(virtualCPUCount, VZVirtualMachineConfiguration.minimumAllowedCPUCount)
        virtualCPUCount = min(virtualCPUCount, VZVirtualMachineConfiguration.maximumAllowedCPUCount)

        return virtualCPUCount
    }
```

メモリ容量
```
    static func computeMemorySize() -> UInt64 {
        # ここでは4GBに設置しているが変更可能
        // Set the amount of system memory to 4 GB; this is a baseline value
        // that you can change depending on your use case.
        var memorySize = (4 * 1024 * 1024 * 1024) as UInt64

        # ゲストOSで利用できる最小値と最大値の処理
        memorySize = max(memorySize, VZVirtualMachineConfiguration.minimumAllowedMemorySize)
        memorySize = min(memorySize, VZVirtualMachineConfiguration.maximumAllowedMemorySize)

        return memorySize
    }
```

```
    static func createBootLoader() -> VZMacOSBootLoader {
        return VZMacOSBootLoader()
    }
```

ディスプレイ
```
    static func createGraphicsDeviceConfiguration() -> VZMacGraphicsDeviceConfiguration {
        # `VZMacGraphicsDeviceConfiguration` のプロパティは`displays`のみ。
        let graphicsConfiguration = VZMacGraphicsDeviceConfiguration()
        
        # 適当に 1920x1200 に設定。
        graphicsConfiguration.displays = [
            // The system arbitrarily chooses the resolution of the display to be 1920 x 1200.
            VZMacGraphicsDisplayConfiguration(widthInPixels: 1920, heightInPixels: 1200, pixelsPerInch: 80)
        ]

        return graphicsConfiguration
    }
```

ディスク
```
    static func createBlockDeviceConfiguration() -> VZVirtioBlockDeviceConfiguration {
        guard let diskImageAttachment = try? VZDiskImageStorageDeviceAttachment(url: diskImageURL, readOnly: false) else {
            fatalError("Failed to create Disk image.")
        }
        let disk = VZVirtioBlockDeviceConfiguration(attachment: diskImageAttachment)
        return disk
    }
```

`VZDiskImageStorageDeviceAttachment` のプロパティは4つ。

- `var url: URL`
  - The URL of the underlying disk image.
- `var isReadOnly: Bool`
  - A Boolean value that indicates whether the underlying disk image is read-only.
- `var cachingMode: VZDiskImageCachingMode`
  - The current cacheing mode for the virtual disk image.
- `var synchronizationMode: VZDiskImageSynchronizationMode`
  - The mode in which the disk image synchronizes data with the underlying storage device.

`VZDiskImageCachingMode` enum の case は 以下。細かい説明はないためキャッシュの挙動は不明。

- `case automatic`
  - Allows the virtualization framework to automatically determine whether to enable data caching.
- `case cached`
  - Enables data caching.
- `case uncached`
  - Disables data caching.

`VZVirtioBlockDeviceConfiguration` のプロパティは一つのみ。init で attachment を指定する。
- `var blockDeviceIdentifier: String`
  The string that identifies the VIRTIO block device.

ネットワーク
```
    static func createNetworkDeviceConfiguration() -> VZVirtioNetworkDeviceConfiguration {
        let networkDevice = VZVirtioNetworkDeviceConfiguration()
        networkDevice.macAddress = VZMACAddress(string: "d6:a7:58:8e:78:d4")!

        let networkAttachment = VZNATNetworkDeviceAttachment()
        networkDevice.attachment = networkAttachment

        return networkDevice
    }
```

プロパティは２つ（親クラスの VZNetworkDeviceConfiguration から継承）。Mac アドレスは何を設定するのが良い？

- `var attachment: VZNetworkDeviceAttachment?`
  - The object that defines how the virtual network device communicates with the host system.
- `var macAddress: VZMACAddress`
  - The media access control (MAC) address to assign to the network device.


`VZNATNetworkDeviceAttachment` は以下の説明。
> A device that routes network requests through the host computer and performs network address translation on the resulting packets.

ホストマシンのネットワークで NAT して通信するということ？


トラックパッド（macOS 13 から利用可能になったらしい?）
```
    static func createPointingDeviceConfiguration() -> VZPointingDeviceConfiguration {
        return VZMacTrackpadConfiguration()
    }
```

キーボード
```
    static func createKeyboardConfiguration() -> VZKeyboardConfiguration {
        if #available(macOS 14.0, *) {
            return VZMacKeyboardConfiguration()
        } else {
            return VZUSBKeyboardConfiguration()
        }
    }
}
```

### `VZMacOSInstaller`


リストアイメージをVMにインストールする。
```
   private func startInstallation(restoreImageURL: URL) {
        let installer = VZMacOSInstaller(virtualMachine: virtualMachine, restoringFromImageAt: restoreImageURL)

        NSLog("Starting installation.")
        installer.install(completionHandler: { (result: Result<Void, Error>) in
            if case let .failure(error) = result {
                fatalError(error.localizedDescription)
            } else {
                NSLog("Installation succeeded.")
            }
        })

        // Observe installation progress.
        installationObserver = installer.progress.observe(\.fractionCompleted, options: [.initial, .new]) { (progress, change) in
            NSLog("Installation progress: \(change.newValue! * 100).")
        }
    }
```
