# iOS KeyChian相关

> 参考资料：[博客园-wuwenhui99](http://www.cnblogs.com/i-o-s-/p/5840625.html)
>

## iOS KeyChain使用和封装，唯一标识符存储

> 参考资料：[简书-广州_虾](https://www.jianshu.com/p/e92e4e4b41a5)
>
> 作者：广州_虾
> 链接：[https://www.jianshu.com/p/e92e4e4b41a5](https://www.jianshu.com/p/e92e4e4b41a5)
> 來源：简书
> 简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。
>

首先一定要在Capabilities中打开Keychain，如图：

![操作步骤](https://upload-images.jianshu.io/upload_images/3346554-a442c658777a58d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

不打开的话进行Keychain操作将会返回一个34018的错误码

引入 __import <Security/Security.h>__

Keychian存储流程如下：

![Keychian存储流程](https://upload-images.jianshu.io/upload_images/3346554-8dba4c98cd94ba5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/570)

__实现方式简要说明：__
如果要存储一个password，需要先遍历Keychain，看它的password是否已经存在于Keychain中，存在的话就更新它的值，不存在就存储。

苹果提供的操作Keychain的方法：

```Objective-C
// 查询
OSStatus SecItemCopyMatching(CFDictionaryRef query, CFTypeRef *result);

// 添加
OSStatus SecItemAdd(CFDictionaryRef attributes, CFTypeRef *result);

// 更新
OSStatus SecItemUpdate(CFDictionaryRef query, CFDictionaryRef attributesToUpdate);

// 删除
OSStatus SecItemDelete(CFDictionaryRef query)
```

操作Keychain常用的key-value

```Objective-C
kSecClass: 有五个值，分别为
  kSecClassGenericPassword (通用密码－－也是接下来使用的)、
  kSecClassInternetPassword (互联网密码)、
  kSecClassCertificate (证书)、
  kSecClassKey (密钥)、
  kSecClassIdentity (身份)

kSecAttrService: 服务
kSecAttrServer: 服务器域名或IP地址
kSecAttrAccount: 账号
kSecAttrAccessGroup: 可以在应用之间共享keychain中的数据
kSecMatchLimit: 返回搜索结果
  kSecMatchLimitOne（一个）、
  kSecMatchLimitAll（全部）
```

这是基于CoreFundation框架的操作，通过带有以上key－value值的字典进行操作，所以写起来还是有些繁琐的。所幸苹果提供了一个源码[Demo](https://developer.apple.com/library/archive/samplecode/GenericKeychain/Introduction/Intro.html#//apple_ref/doc/uid/DTS40007797)。demo中使用service和account以及accessGroup进行password的存储，也就是一个kSecClassGenericPassword类型的Keychain表，一个account对应一个password，可以很方便在APP中进行Key-Value类型的存储。

不过由于它是使用Swift编写的，在我的小项目中不适合引入混编，所以我就按着demo 的逻辑抄了个OC版的出来。个人看来，OC的还是好懂点，去除了各种try catch，思路上会清晰很多。

__代码片段说明：__
下面进行一些代码片段的说明

* 初始化方法：

```Objective-C
- (instancetype)initWithSevice:(NSString *)service account :(NSString *)account accessGroup:(NSString *)accessGroup {
    if (self = [super init]) {
        _service = service;
        _account = account;
        _accessGroup = accessGroup;//可以在不同的app当中共享
    }
    return self;
}
```

* 主要方法

```Objective-C
- (void)savePassword:(NSString *)password;

- (BOOL)deleteItem;

- (NSString *)readPassword;

// 返回当前accessGroup下的service的所有Keychain Item
+ (NSArray *)passwordItemsForService:(NSString *)service accessGroup:(NSString *)accessGroup;
```

* 使用示例

```Objective-C
// 存储

KeychainWrapper *wrapper = [[KeychainWrapper alloc] initWithSevice:kKeychainService account:self.accountField.text accessGroup:kKeychainAccessGroup];

[wrapper savePassword:self.passwordField.text];

// 读取当前account 的password：
KeychainWrapper *item = [[KeychainWrapper alloc] initWithSevice:service account:acount accessGroup:accessGroup];

[item readPassword];

// 返回当前accessGroup下的service的所有Keychain：

NSArray *keychains = [KeychainWrapper passwordItemsForService:kKeychainService accessGroup:kKeychainAccessGroup];
```

踩坑：OSStatus状态错误码：

50: 请求参数错误。我就曾经把kSecAttrService写成了kSecAttrServer，crash到哭了。。。
34018:前面说的Capabilities...

完整代码，如下：

```Objective-C
- (void)savePassword:(NSString *)password {
    NSData *encodePasswordData = [password dataUsingEncoding:NSUTF8StringEncoding];

    // if password was existed, update
    NSString *originPassword = [self readPassword];

    if (originPassword.length > 0) {
        NSMutableDictionary *updateAttributes = [NSMutableDictionary dictionary];
        updateAttributes[(__bridge id)kSecValueData] = encodePasswordData;

        NSMutableDictionary *query = [self keychainQueryWithService:_service account:_account accessGroup:_accessGroup];
        OSStatus statusCode = SecItemUpdate(
                                           (__bridge CFDictionaryRef)query,
                                           (__bridge CFDictionaryRef)updateAttributes);
        NSAssert(statusCode == noErr, @"Couldn't update the Keychain Item." );
    }else{
        // else , add
        NSMutableDictionary *attributes = [self keychainQueryWithService:_service account:_account accessGroup:_accessGroup];
        attributes[(__bridge id)kSecValueData] = encodePasswordData;

        OSStatus status = SecItemAdd((__bridge CFDictionaryRef)attributes, nil);

        NSAssert(status == noErr, @"Couldn't add the Keychain Item.");
    }
}

- (NSString *)readPassword {
    NSMutableDictionary *attributes = [self keychainQueryWithService:_service account:_account accessGroup:_accessGroup];
    attributes[(__bridge id)kSecMatchLimit] = (__bridge id)(kSecMatchLimitOne);
    attributes[(__bridge id)kSecReturnAttributes] = (__bridge id _Nullable)(kCFBooleanTrue);
    attributes[(__bridge id)kSecReturnData] = (__bridge id _Nullable)(kCFBooleanTrue);

    CFMutableDictionaryRef queryResult = nil;
    OSStatus keychainError = noErr;
    keychainError = SecItemCopyMatching((__bridge CFDictionaryRef)attributes,(CFTypeRef *)&queryResult);
    if (keychainError == errSecItemNotFound) {
        if (queryResult) CFRelease(queryResult);
        return nil;
    } else if (keychainError == noErr) {

        if (queryResult == nil){return nil;}

        NSMutableDictionary *resultDict = (__bridge NSMutableDictionary *)queryResult;
        NSData *passwordData = resultDict[(__bridge id)kSecValueData];

        NSString *password = [[NSString alloc] initWithData:passwordData encoding:NSUTF8StringEncoding];

        return password;
    }else
    {
        NSAssert(NO, @"Serious error.\n");
        if (queryResult) CFRelease(queryResult);
    }

    return nil;
}

- (NSMutableDictionary *)keychainQueryWithService:(NSString *)service account:(NSString *)account accessGroup:(NSString *)accessGroup {

    NSMutableDictionary *query = [NSMutableDictionary dictionary];

    query[(__bridge id)kSecClass] = (__bridge id)kSecClassGenericPassword;

    query[(__bridge id)kSecAttrService] = service;

    query[(__bridge id)kSecAttrAccount] = account;

    query[(__bridge id)kSecAttrAccessGroup] = accessGroup;

    return query;
}

+ (NSArray *)passwordItemsForService:(NSString *)service accessGroup:(NSString *)accessGroup {
    NSMutableDictionary *query = [[[self alloc]init] keychainQueryWithService:service account:nil accessGroup:accessGroup];

    query[(__bridge id)kSecMatchLimit] = (__bridge id)kSecMatchLimitAll;
    query[(__bridge id)kSecReturnAttributes] = (__bridge id _Nullable)(kCFBooleanTrue);
    query[(__bridge id)kSecReturnData] = (__bridge id _Nullable)(kCFBooleanFalse);

    CFMutableArrayRef queryResult = nil;
    OSStatus status = noErr;
    status = SecItemCopyMatching((__bridge CFDictionaryRef)query,(CFTypeRef *)&queryResult);

    if (status == errSecItemNotFound) {
        return @[];
    } else if (status == noErr) {

        NSArray<NSDictionary *> *resultArray = (__bridge NSArray<NSDictionary *> *)queryResult;

        if (resultArray.count == 0){return @[];}

        NSMutableArray *passwordItems = [NSMutableArray array];

        for (NSDictionary *result in resultArray) {
            NSString *acount = result[(__bridge id)kSecAttrAccount];

            if (acount.length > 0) {
                KeychainWrapper *item = [[KeychainWrapper alloc] initWithSevice:service account:acount accessGroup:accessGroup];
                [passwordItems addObject:item];
            }
        }

        return passwordItems;
    }else {
        NSAssert(NO, @"Serious error.\n");
        if (queryResult) CFRelease(queryResult);
        return @[];
    }
}

- (BOOL)deleteItem {
    // Delete the existing item from the keychain.
    NSMutableDictionary *query = [self keychainQueryWithService:self.service account:self.account accessGroup:self.accessGroup];

    OSStatus status = SecItemDelete((__bridge CFDictionaryRef)query);

    if (status != noErr && status != errSecItemNotFound) {
        return NO;
    }
    return true;
}
```

> 结尾 [苹果官方文档：Keychain Services](https://developer.apple.com/documentation/security/keychain_services?language=objc)