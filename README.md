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
	SEND_OUT_WAIT_RETURN,	//wait_tick_cnt++,当达到第一次触发值时，发送“ok\n"一次，以防丢失！达第二次时，再发送一次"ok\n"，达到第三次时，状态设为SEND_FAIL
	SEND_FAIL,				//进入到此，说明发送已失败，检测若蓝牙还在连接，则每二秒钟再发送一次”OK\n“，若蓝牙没连接，则将状态设为SEND_DISABLE;
	break;
}SEND_STATUS_t;	

typedef struct
{
	uint32_t wait_tick_cnt; //发送“ok\n"后，若接收延时，用此计数，可以再2次允许
	SEND_STATUS_t status;	//状态控制
}rx_ctr_t;	
 
启动一个30ms的软件定时器，它是连接成功时启动，断开时关闭。它调用void uart_ctr_handle(rx_ctr_t *ctr)函数。
void uart_ctr_handle(rx_ctr_t *ctr)
{
	switch(ctr->status)
	{
		case SEND_DISABLE:
		break;
		
		SEND_DONE:
		SEND_ENABLE:
			ctr->wait_tick_cnt = 0;
			uart_tx("ok\n");
			ctr->status = SEND_OUT_WAIT_RETURN;
		break;
		
		SEND_OUT_WAIT_RETURN:
			ctr->wait_tick_cnt++;
			if(ctr->wait_tick_cnt < 4)
			{
				uart_tx("ok\n");
			}
			else if(ctr->wait_tick_cnt < 7)
			{
				uart_tx("ok\n");
			}
			else if(ctr->wait_tick_cnt >= 7)
			{
				ctr->status = SEND_FAIL;
				ctr->wait_tick_cnt = 0;
			}
		break;
		
		SEND_FAIL:
			if( m_conn_handle != BLE_CONN_HANDLE_INVALID)
			{
				ctr->wait_tick_cnt++;
				if(ctr->wait_tick_cnt >= 600)
				{
					ctr->wait_tick_cnt = 0;
					uart_tx("ok\n");
				}
				else
				{
					ctr->wait_tick_cnt = 0;
					ctr->status = SEND_DISABLE;
				}
			}	
		break;
		
		default：
		break;
	}
} 


