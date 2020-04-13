# CCNet  BlueTooth蓝牙SDK

使用SDK前需导入libCCNetBluetooth.a 及 BabyBluetooth.h头文件到工程



## iOS SDK 文档说明

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


//设置查找设备的过滤器
[baby setFilterOnDiscoverPeripherals:^BOOL(NSString *peripheralName, NSDictionary *advertisementData, NSNumber *RSSI) {
    //最常用的场景是查找某一个前缀开头的设备
   if ([peripheralName hasPrefix:@"TM"] ) {
      return YES;
   }
     return NO;
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





Android SDK文档说明(敬请期待)
