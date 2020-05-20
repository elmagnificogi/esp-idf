# Espressif

这个是我自己用的esp-idf，稍微基于[NathanReeves](https://github.com/NathanReeves)/**[esp-idf](https://github.com/NathanReeves/esp-idf)**的修改了一下，适配中文环境，增加搭建脚本

- 实际上官方的HID已经添加了,但是目前只有BLE的device hid,而不支持Classic的hid,已经提了issue,官方回复还要等待一段时间增加老蓝牙的hid支持和demo

等官方HID出了以后,就切到官方版本



[Molorius/esp-idf](https://github.com/Molorius/esp-idf)

- 已经问过原作者,不再继续更新或者修复他的hid版本了

## 已知bug

### 内存泄漏

已知：由于NS固定每接收到12条信息回一次震动信息，而震动信息基于[Molorius/esp-idf](https://github.com/Molorius/esp-idf)在解包过程中会固定造成内存泄漏-32字节/次，最终导致内存不够，程序挂掉.



该问题暂时无解,只能通过track操作让每次收到的震动信息从底层就丢弃掉,从而不会造成内存泄漏.



具体泄漏点应该位于下面的流程中,具体在哪里,由于缺少调试器,靠打印输出非常麻烦,所以放弃了

```c
   the rumble data is acl_data,it came from host_recv_pkt_cb call many lower funcs to
       
   l2c_csm_execute (p_ccb, L2CEVT_L2CAP_DATA, p_msg);

   l2c_csm_open (p_ccb, event, p_data);

   then
       
       case L2CEVT_L2CAP_DATA:
       (*p_ccb->p_rcb->api.pL2CA_DataInd_Cb)(p_ccb->local_cid, (BT_HDR *)p_data);

   ....(there is some call i cant find it now)
       
   then
       
   btc_hd_upstreams_evt()
       
       case BTA_HD_INTR_DATA_EVT:
          HAL_CBACK(bt_hd_callbacks, intr_data_cb, p_data->intr_data.report_id,
                p_data->intr_data.len, p_data->intr_data.p_data);
	// call the user callback. after it, it free the msg.arg in btc_task();
	// then the heap less 32 bytes
```

### 手柄配对缺少配对命令

理论上配对时,应该会发送subcmd:0x01 0x04开头的请求配对信息,但是实际上并不会,应该是被这个魔改的HID屏蔽了或者丢弃了,导致无法收到这个命令,暂时无解



