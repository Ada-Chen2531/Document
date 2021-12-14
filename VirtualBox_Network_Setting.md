### Windows 開發時 ViturlBox 網路卡設定

* 《僅限主機》的意思，是表示這台VM是在封閉環境的主機，與外部網路隔離

* 在此我們會使用兩張網卡
> 一張設定成 `僅限主機`  
>> * 用於 VS Code 與本機電腦內部 SSH 連線用  
>> * 因此網路不穩定或是斷線時，不會對 SSH 造成影響)  

> 一張設定成 `NAT`
>> * 用於 AWS VPN 連線時，VM 的網際網路會跟著 VPN  
>> * VM 透過本機電腦來存取網際網路，因此本機電腦網路斷線時 VM 也無法存取網際網路

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/VM_network_1.jpg)

![Image](https://github.com/Ada-Chen2531/Document/raw/main/Pictures/VM_network_2.jpg)
