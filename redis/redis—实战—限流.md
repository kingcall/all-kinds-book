- 限流是对系统的出入流量进行控制，防止大流量出入，导致资源不足，系统不稳定。

![image-20201125212433027](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:24:36-21:24:33-image-20201125212433027.png)

### 数据结构

![image-20201125212501691](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:25:02-image-20201125212501691.png)
- 限流系统的实现是基于 redis 的，本可以和应用无关，但是为了做限流元数据配置的统一管理，按应用维度管理和使用，在数据结构中加入了 apps 这个字段，出现问题，排查起来也比较方便。

### 管理界面

![image-20201125212517831](https://kingcall.oss-cn-hangzhou.aliyuncs.com/blog/img/2020/11/25/21:25:18-image-20201125212517831.png)

