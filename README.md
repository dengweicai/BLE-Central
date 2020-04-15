# CCNet  BlueTooth蓝牙SDK

此文档用于说明 Central模式(App端) SDK使用





## iOS SDK 文档说明

使用SDK前需导入libCCNetBluetooth.a 及 BabyBluetooth.h头文件到工程



1.引入.h文件

```objective-c
#import "BabyBluetooth.h"
```



2.初始化蓝牙库

```objective-c
BabyBluetooth *baby;
baby = [BabyBluetooth shareBabyBluetooth];
```



3.设置扫描设备回调，当扫描成功后保存peripheral对象，以便下一步连接设备。当然也可以通过setFilterOnDiscoverPeripherals 方法委托过滤器扫描设备名称查找目标设备。

```objective-c
[baby setBlockOnCentralManagerDidUpdateState:^(CBCentralManager *central) {
    if (central.state == CBCentralManagerStatePoweredOn) {
				//设备打开成功，开始扫描设备     
    }
 }];


[baby setBlockOnDiscoverToPeripherals:^(CBCentralManager *central, CBPeripheral *peripheral, NSDictionary *advertisementData, NSNumber *RSSI) {
        NSLog(@"搜索到了设备:%@",peripheral.name);
}];
```



4.查找到设备后开始进行设备连接

```objective-c
//使用不同的channel切换委托回调,channelOnPeropheralString 为任意唯一字符串标识
[baby setBlockOnConnectedAtChannel:channelOnPeropheralString block:^(CBCentralManager *central, CBPeripheral *peripheral) {
     //设备连接成功
}];

[baby setBlockOnFailToConnectAtChannel:channelOnPeropheralView block:^(CBCentralManager *central, CBPeripheral *peripheral, NSError *error) {
		//设备连接失败
}];


[baby setBlockOnDisconnectAtChannel:channelOnPeropheralView block:^(CBCentralManager *central, CBPeripheral *peripheral, NSError *error) {
     //设备断开连接
}];

baby.having(self.currPeripheral).and.channel(channelOnPeropheralView).then.connectToPeripherals()
```



5.设备连接成功后，开始读取Characteristics

```objective-c
[baby setBlockOnDiscoverCharacteristics:^(CBPeripheral *peripheral, CBService *service, NSError *error) {
     NSLog(@"service name:%@",service.UUID);
     for (CBCharacteristic *c in service.characteristics) {
        NSLog(@"charateristic name is :%@",c.UUID);
     }
}];

baby.having(self.currPeripheral).and.channel(channelOnPeropheralString).then.connectToPeripherals().discoverServices().discoverCharacteristics().begin()
```



6.获取到charateristics之后我们可以获取charateristics的值或者订阅它的通知

(1)读值

```objective-c
[baby setBlockOnReadValueForCharacteristicAtChannel:channelOnCharacteristicView block:^(CBPeripheral *peripheral, CBCharacteristic *characteristics, NSError *error) {
		NSLog(@"characteristic name:%@ value is:%@",characteristics.UUID,characteristics.value);
}];
//读取服务
baby.channel(channelOnPeropheralString).characteristicDetails(self.currPeripheral,self.characteristic);
```

(2)订阅通知

```objective-c
if (self.characteristic.properties & CBCharacteristicPropertyNotify || self.characteristic.properties & CBCharacteristicPropertyIndicate) {
        
		if(self.characteristic.isNotifying) {//取消通知
    		[baby cancelNotify:self.currPeripheral characteristic:self.characteristic];
      
  	}else{//订阅通知
      [weakSelf.currPeripheral setNotifyValue:YES forCharacteristic:self.characteristic];
      [baby notify:self.currPeripheral  characteristic:self.characteristic
     block:^(CBPeripheral *peripheral, CBCharacteristic *characteristics, NSError *error) {
						NSLog(@"new value %@",characteristics.value);
			}];
   }
}
```







## Android SDK文档说明

使用SDK前导入bluetoothkit.jar至工程，配AndroidManifest.xml文件



1.创建一个BluetoothClient，建议作为一个全局单例，管理所有BLE设备的连接。

```java
BluetoothClient mClient = new BluetoothClient(context);
```



2.支持经典蓝牙和BLE设备混合扫描，可自定义扫描策略。每次扫描都要创建新的SearchRequest，不能复用。如果

```java
SearchRequest request = new SearchRequest.Builder()
        .searchBluetoothLeDevice(3000, 3)   // 先扫BLE设备3次，每次3s
        .build();

mClient.search(request, new SearchResponse() {
    @Override
    public void onSearchStarted() {

    }

    @Override
    public void onDeviceFounded(SearchResult device) {
        Beacon beacon = new Beacon(device.scanRecord);
        BluetoothLog.v(String.format("beacon for %s\n%s", device.getAddress(), beacon.toString()));
    }

    @Override
    public void onSearchStopped() {

    }

    @Override
    public void onSearchCanceled() {

    }
});
```

如果需要手动停止扫描调用

```java
mClient.stopSearch();
```



3.扫描到设备后，对设备进行连接

```java
//如果要监听蓝牙连接状态可以注册回调，只有两个状态：连接和断开。
mClient.registerConnectStatusListener(MAC, mBleConnectStatusListener);

private final BleConnectStatusListener mBleConnectStatusListener = new BleConnectStatusListener() {

    @Override
    public void onConnectStatusChanged(String mac, int status) {
        if (status == STATUS_CONNECTED) {

        } else if (status == STATUS_DISCONNECTED) {

        }
    }
};
mClient.unregisterConnectStatusListener(MAC, mBleConnectStatusListener);


//设置连接参数
BleConnectOptions options = new BleConnectOptions.Builder()
        .setConnectRetry(3)   // 连接如果失败重试3次
        .setConnectTimeout(30000)   // 连接超时30s
        .setServiceDiscoverRetry(3)  // 发现服务如果失败重试3次
        .setServiceDiscoverTimeout(20000)  // 发现服务超时20s
        .build();
//开始进行连接
mClient.connect(MAC, options, new BleConnectResponse() {
    @Override
    public void onResponse(int code, BleGattProfile data) {

    }
});

//断开连接
mClient.disconnect(MAC);
```



4.连接成功后从BleGattProfile data 读取serviceUUID、characterUUID，就可以开始读数据了



（1）读取数据

```
mClient.read(MAC, serviceUUID, characterUUID, new BleReadResponse() {
    @Override
    public void onResponse(int code, byte[] data) {
        if (code == REQUEST_SUCCESS) {

        }
    }
});
```

（2）订阅通知

```java
//订阅通知
mClient.notify(MAC, serviceUUID, characterUUID, new BleNotifyResponse() {
    @Override
    public void onNotify(UUID service, UUID character, byte[] value) {
        
    }

    @Override
    public void onResponse(int code) {
        if (code == REQUEST_SUCCESS) {

        }
    }
});


//关闭通知
mClient.unnotify(MAC, serviceUUID, characterUUID, new BleUnnotifyResponse() {
    @Override
    public void onResponse(int code) {
        if (code == REQUEST_SUCCESS) {

        }
    }
});
```

