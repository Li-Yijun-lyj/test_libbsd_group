#include <rtems.h>
#include <rtems/bsd/bsd.h>
#include <bsp.h>
#include <assert.h>
#include <stdio.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>

#define RTEMS_BSD_ARGC(array) (sizeof(array) / sizeof((array)[0]) - 1)
#define ARGC(x) RTEMS_BSD_ARGC(x)

rtems_task Init(rtems_task_argument ignored) {
    int sock, ret, opt;
    struct sockaddr_in addr;
    const char *msg = "RTEMS broadcast message";

    // 初始化 BSD 网络栈并运行 /etc/rc.conf
    rtems_status_code sc = rtems_bsd_initialize();
    assert(sc == RTEMS_SUCCESSFUL);

    char *vlan_ip[] = {
		"ifconfig",
		"cgem0",
        "inet",
		"169.254.1.20 up/24",
        "netmask",
		"255.255.0.0",
		NULL
	};
    char *dflt_route[] = {
		"route",
		"add",
		"default",
		"192.168.96.1",
		NULL
	};
    rtems_bsd_command_ifconfig(RTEMS_BSD_ARGC(vlan_ip),vlan_ip);
    rtems_bsd_command_route(ARGC(dflt_route), dflt_route);

    // 创建 UDP 套接字
    sock = socket(AF_INET, SOCK_DGRAM, IPPROTO_UDP);
    if (sock < 0) {
        perror("socket");
        return;
    }

    // 启用广播选项
    opt = 1;
    if (setsockopt(sock, SOL_SOCKET, SO_BROADCAST, &opt, sizeof(opt)) < 0) {
        perror("setsockopt");
        return;
    }

    // 配置目标广播地址和端口
    memset(&addr, 0, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(12345);
    addr.sin_addr.s_addr = inet_addr("255.255.0.0");

    // 循环发送广播消息
    while (1) {
        ret = sendto(sock, msg, strlen(msg), 0,
                     (struct sockaddr *)&addr, sizeof(addr));
        if (ret < 0) {
            perror("sendto");
        } else {
            printf("Sent %d bytes\n", ret);
        }
        rtems_task_wake_after(RTEMS_MILLISECONDS_TO_TICKS(1000));
    }
}
/*
 * Simple RTEMS configuration
 */

#define CONFIGURE_APPLICATION_NEEDS_CLOCK_DRIVER
#define CONFIGURE_APPLICATION_NEEDS_CONSOLE_DRIVER

#define CONFIGURE_UNLIMITED_OBJECTS
#define CONFIGURE_UNIFIED_WORK_AREAS

#define CONFIGURE_RTEMS_INIT_TASKS_TABLE

#define CONFIGURE_INIT

#include <rtems/confdefs.h>
