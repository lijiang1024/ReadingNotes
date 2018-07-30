# 获取设备的型号

> 获得Apple设备型号 [https://www.theiphonewiki.com/wiki/Models](https://www.theiphonewiki.com/wiki/Models)

方法一：

```Objetive-C
// 需要 #import "sys/utsname.h"

struct utsname systemInfo;
uname(&systemInfo);
NSString *platform = [NSString stringWithCString:systemInfo.machine encoding:NSUTF8StringEncoding];

//iPhone
if ([platform isEqualToString:@"iPhone1,1"])    return @"iPhone 1G";
```

方法二：

```Objetive-C
// 需要 #include <sys/sysctl.h>

int mib[2];
size_t len;
char *machine;

mib[0] = CTL_HW;
mib[1] = HW_MACHINE;
sysctl(mib, 2, NULL, &len, NULL, 0);
machine = malloc(len);
sysctl(mib, 2, machine, &len, NULL, 0);

NSString *platform = [NSString stringWithCString:machine encoding:NSASCIIStringEncoding];
free(machine);

//iPhone
if ([platform isEqualToString:@"iPhone1,1"])    return @"iPhone 1G";

if ([platform isEqualToString:@"i386"])   return @"iPhone Simulator";
if ([platform isEqualToString:@"x86_64"]) return @"iPhone Simulator";
```