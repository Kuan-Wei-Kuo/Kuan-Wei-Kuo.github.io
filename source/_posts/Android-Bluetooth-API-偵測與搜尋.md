---
title: Android. Bluetooth API 偵測與搜尋
date: 2016-06-20 10:58:55
tags: Android
---

藍芽裝置，最初的步驟就是搜尋附近的裝置，這其實已經是早期的技術了，但到現在還是被廣泛使用，尤其對於物聯網來說，是非常方便的，因為目前幾乎每一支手機都有藍芽，如果產品想要串接藍芽，其實也不難，但規格要盡量一致才不會出現奇怪的問題。

通常藍芽會直接在Activity中使用，因為我們需要搭配BroadcastReceiver來做接收搜尋等動作，因此我們需要在Activtiy中建立，當然如果想要使用OOP這種寫法也可以，也只是將Code的位置擺放去其他地方。

# Step 1. 加入Permission
```xml
<uses-permission android:name="android.permission.BLUETOOTH" />
<uses-permission android:name="android.permission.BLUETOOTH_ADMIN" />
```

# Step 2. 偵測手機是否支援藍芽
```java
mBluetoothAdapter = BluetoothAdapter.getDefaultAdapter();
if(mBluetoothAdapter == null)
    bluetoothList.setText("您的裝置沒有支援藍芽");
```
BluetoothAdapter我們是不需要Intance的，因為這是早已經存在的東西，手機本身就帶有Intance的BluetoothAdapter，當然很簡單的，我們如果是NULL的，很明顯裝置並沒有藍芽。

# Step 3. 查詢藍芽是否有開啟
```java
if (!mBluetoothAdapter.isEnabled()) {
    Intent enableIntent = new Intent(BluetoothAdapter.ACTION_REQUEST_ENABLE);
    startActivityForResult(enableIntent, REQUEST_ENABLE_BT);
} 
```
當開啟藍芽界面並完成選擇動作時，ActivityForResult這個Method就會執行，也將會回傳REQUEST_CODE，讓開發者可以判斷。對了，這一段Code建議寫在onStart內，因為我們onCreate其實不太適合做這種動作，有疑問的可以在去看看Acivity週期。

# Step 4. 搜尋藍芽裝置
```java
if (mBluetoothAdapter.isDiscovering())
    mBluetoothAdapter.cancelDiscovery();
mBluetoothAdapter.startDiscovery();
```
如果mBluetoothAdapter正在搜尋，但使用者又點了一次按鈕之類的，我們得取消查詢再次展開查詢的動作，不然可能會造成Loop的問題。

# Step 5. 接收藍芽裝置
```java
IntentFilter filter = new IntentFilter(BluetoothDevice.ACTION_FOUND);
this.registerReceiver(mReceiver, filter);

filter = new IntentFilter(BluetoothAdapter.ACTION_DISCOVERY_FINISHED);
this.registerReceiver(mReceiver, filter);

private ArrayAdapter<String> mNewDevicesArrayAdapter;

private final BroadcastReceiver mReceiver = new BroadcastReceiver() {
    @Override
    public void onReceive(Context context, Intent intent) {
        String action = intent.getAction();
        if (BluetoothDevice.ACTION_FOUND.equals(action)) {
            BluetoothDevice device = intent.getParcelableExtra(BluetoothDevice.EXTRA_DEVICE);
            if (device.getBondState() != BluetoothDevice.BOND_BONDED) {
                mNewDevicesArrayAdapter.add(device.getName() + "\n" + device.getAddress());
            }
        } else if (BluetoothAdapter.ACTION_DISCOVERY_FINISHED.equals(action)) {
            if (mNewDevicesArrayAdapter.getCount() == 0) {
                String noDevices = "No Devices";
                mNewDevicesArrayAdapter.add(noDevices);
            }
        }
    }
};
```

BluetoothAdapter與BluetoothDevice有提供一些StringAction，但通常都使用FOUND與FINISHED兩種為多，在搭配Receiver接收每筆所查詢到的藍芽裝置。