---
layout: post
title:  "Bluetoothテザリング"
date:   2017-10-13 15:26:12 +0900
categories: コンピュータ
---

# いつも車内で慌てて設定しようとして、トンネル突入

まえもってテストしとけよコンチクショウめ。

[Bluetoothctl][Bluetoothctl] とか
[Bluetoothでテザリング][BluetoothTethering] あたりを参照。

手順は以下の通り

1. Android端末をBluetoothテザリングに設定する
2. Linuxからandroid端末をペアリング
3. dbus-sendコマンドを使ってネットワークインターフェイスを作成
4. dhcpcdコマンドでネットワーク接続を設定する

## ペアリング

    % bluetoothctl
    [bluetooth] # list
    [bluetooth] # devices
    [bluetooth] # power on
    [bluetooth] # scan on
    [bluetooth] # devices
    ...
    Device CC:F3:A5:C1:D6:B4 Nextbit Robin
    ...
    [bluetooth] # pair CC:F3:A5:C1:D6:B
    Request confirmation
    [agent] Confirm passkey 447317 (yes/no): yes
    ...
    Pairing successful
    [bluetooth] # trust CC:F3:A5:C1:D6:B4
    [CHG] Device CC:F3:A5:C1:D6:B4 Trusted: yes
    Changing CC:F3:A5:C1:D6:B4 trust succeeded
    [bluetooth] # quit

初回は以上の手順だったが、次回からはさくっと繋がるはず

## ネットワークインターフェイスを作成と接続
    % dbus-send --system --type=method_call --dest=org.bluez /org/bluez/hci0/dev_CC_F3_A5_C1_D6_B4 org.bluez.Network1.Connect string:'nap'
    % ip addr
    ...
    5: bnep0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
         link/ether 28:b2:bd:2b:65:50 brd ff:ff:ff:ff:ff:ff
         inet6 fe80::2ab2:bdff:fe2b:6550/64 scope link
         valid_lft forever preferred_lft forever
    % sudo dhcpcd bnep0
    DUID 00:01:00:01:1d:d0:8f:ea:28:b2:bd:2b:65:4c
    bnep0: IAID bd:2b:65:50
    bnep0: soliciting an IPv6 router
    bnep0: soliciting a DHCP lease
    bnep0: offered 192.168.44.109 from 192.168.44.1
    bnep0: probing address 192.168.44.109/24
    bnep0: leased 192.168.44.109 for 3600 seconds
    bnep0: adding route to 192.168.44.0/24
    bnep0: adding default route via 192.168.44.1
    forked to background, child pid 1854

    % ip addr
    ...
    5: bnep0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UNKNOWN group default qlen 1000
         link/ether 28:b2:bd:2b:65:50 brd ff:ff:ff:ff:ff:ff
         inet 192.168.44.109/24 brd 192.168.44.255 scope global bnep0
         valid_lft forever preferred_lft forever
         inet6 fe80::2ab2:bdff:fe2b:6550/64 scope link
         valid_lft forever preferred_lft forever

[Bluetoothctl]: https://wiki.archlinux.jp/index.php/Bluetooth#Bluetoothctl
[BluetoothTethering]: https://wiki.archlinux.jp/index.php/Android_%E3%83%86%E3%82%B6%E3%83%AA%E3%83%B3%E3%82%B0#Bluetooth_.E3.81.A7.E3.83.86.E3.82.B6.E3.83.AA.E3.83.B3.E3.82.B0