# 1
解决串口定时发送造成蓝牙断开的方法：
断开原因是:在ble没互连上时，ble不断接收到串口tx过来的数据，而造溢出,而关闭。
# 2
解决方法：
  串口发给ble数据不能串口自已不停地发！
  串口第一次发送：判断是否连接成功，成功了，就发送特定信息通知串口可以tx了,然后轮询，看是否接收到串口的信息，若没有，再次发送，直到接收到确认信息，
  尝试5次，不然作出错处理。
# 3
BLE RX Part

设定一个枚举变量：SEND_STATUS_t
struct enum
{
	SEND_DISABLE, 			//初始化的状态，不执行,或断开连接时设定
	break;
	SEND_DONE,				//wait_tick_cnt = 0; 状态更改为SEND_ENABLE,它在ble的串口接收发送数据蓝牙后触发
	SEND_ENABLE,			//wait_tick_cnt = 0;通过串口向stm32发送"ok\n",然后状态更改为SEND_OUT_WAIT_RETURN，它由首次连上蓝牙后触发。
	break;
	SEND_OUT_WAIT_RETURN,	//wait_tick_cnt++,当达到第一次触发值时，发送“ok\n"一次，以防丢失！达第二次时，再发送一次"ok\n"，达到第三次时，状态设为SEND_DISABLE
	
}SEND_STATUS_t;				


