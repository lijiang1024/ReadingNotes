# iOS KeyChain使用和封装，唯一标识符存储（使用MRC模式实现）

> 说明：使用此代码需要在 __Build Phases -> Complie Sources__ 中找到该类的m文件 设置 Compiler Flags 为 __-fno-objc-arc__


__实现类代码__

```Objetive-C

#import "ToolKit.h"
#import <Security/Security.h>

// replace the identity with your company's domain
static const char kKeychainUDIDItemIdentifier[]  = "UUID";
static const char kKeyChainUDIDAccessGroup[] = "XJ945DFZ3U.com.dawnpro.ios.demo";

@implementation ToolKit

+ (NSString *)UUID {

  NSString *udid = [ToolKit getUDIDFromKeyChain];
  if (!udid) {
    udid = [ToolKit _identifierForVendor];
    NSLog(@"Keychain 中没有UUID --- 设备广告标识为：%@", udid);
    [ToolKit setUDIDToKeyChain:udid];
  } else {
    NSLog(@"Keychain 中有UUID --- 设备广告标识为：%@", udid);
  }

  return udid;
}

/**
 * 获取设备的广告标识符
 * iOS 7.0
 * Starting from iOS 7, the system always returns the value 02:00:00:00:00:00
 * when you ask for the MAC address on any device.
 * use identifierForVendor + keyChain
 * make sure UDID consistency atfer app delete and reinstall
 */
+ (NSString *)_identifierForVendor {
    return [[UIDevice currentDevice].identifierForVendor UUIDString];
}

#pragma mark - Helper Method for make identityForVendor consistency

//
// 以下的代码实现
// Created by 陈将 on 14/11/13.
// Copyright (c) 2014年 transn. All rights reserved.
//

+ (NSString*)getUDIDFromKeyChain {

    NSMutableDictionary *dictForQuery = [[NSMutableDictionary alloc] init];
    [dictForQuery setValue:(id)kSecClassGenericPassword forKey:(id)kSecClass];

    // set Attr Description for query
    [dictForQuery setValue:[NSString stringWithUTF8String:kKeychainUDIDItemIdentifier] forKey:(id)kSecAttrDescription];

    // set Attr Identity for query
    NSData *keychainItemID = [NSData dataWithBytes:kKeychainUDIDItemIdentifier length:strlen(kKeychainUDIDItemIdentifier)];
    [dictForQuery setObject:keychainItemID forKey:(id)kSecAttrGeneric];

    // The keychain access group attribute determines if this item can be shared
    // amongst multiple apps whose code signing entitlements contain the same keychain access group.
    NSString *accessGroup = [NSString stringWithUTF8String:kKeyChainUDIDAccessGroup];

    if (accessGroup != nil) {
#if TARGET_IPHONE_SIMULATOR
        // Ignore the access group if running on the iPhone simulator.
        //
        // Apps that are built for the simulator aren't signed, so there's no keychain access group
        // for the simulator to check. This means that all apps can see all keychain items when run
        // on the simulator.
        //
        // If a SecItem contains an access group attribute, SecItemAdd and SecItemUpdate on the
        // simulator will return -25243 (errSecNoAccessForItem).
#else
        [dictForQuery setObject:accessGroup forKey:(id)kSecAttrAccessGroup];
#endif
    }

    [dictForQuery setValue:(id)kCFBooleanTrue forKey:(id)kSecMatchCaseInsensitive];
    [dictForQuery setValue:(id)kSecMatchLimitOne forKey:(id)kSecMatchLimit];
    [dictForQuery setValue:(id)kCFBooleanTrue forKey:(id)kSecReturnData];

    OSStatus status = noErr;
    NSData *udidValue = nil;
    NSString *udid = nil;
    status = SecItemCopyMatching((CFDictionaryRef)dictForQuery, (CFTypeRef*)&udidValue);
    NSLog(@"KeyChain query 1 status ========= code: %d  - udidValue : %@", status, udidValue);

//    OSStatus queryErr = noErr;
//    NSMutableDictionary *dict = nil;
//    [dictForQuery setValue:(id)kCFBooleanTrue forKey:(id)kSecReturnAttributes];
//    queryErr = SecItemCopyMatching((CFDictionaryRef)dictForQuery, (CFTypeRef*)&dict);
//
//    NSLog(@"KeyChain query 2 status ========= code: %d  - dict : %@", status, dict);

    if (status == errSecItemNotFound) {
        NSLog(@"KeyChain Item: %@ not found!!!", [NSString stringWithUTF8String:kKeychainUDIDItemIdentifier]);
    } else if (status != errSecSuccess) {
        NSLog(@"KeyChain Item query Error!!! Error code:%d", status);
    } else if (status == errSecSuccess) {
        NSLog(@"KeyChain Item: %@", udidValue);
        if (udidValue) {
            udid = [NSString stringWithUTF8String:udidValue.bytes];
            [udidValue release];
        }
//        [dict release];
    }

    [dictForQuery release];
    return udid;
}

+ (BOOL)setUDIDToKeyChain:(NSString*)udid {

    NSMutableDictionary *dictForAdd = [[NSMutableDictionary alloc] init];

    [dictForAdd setValue:(id)kSecClassGenericPassword forKey:(id)kSecClass];
    [dictForAdd setValue:[NSString stringWithUTF8String:kKeychainUDIDItemIdentifier] forKey:(id)kSecAttrDescription];

    [dictForAdd setValue:@"UUID" forKey:(id)kSecAttrGeneric];

    // Default attributes for keychain item.
    [dictForAdd setObject:@"" forKey:(id)kSecAttrAccount];
    [dictForAdd setObject:@"" forKey:(id)kSecAttrLabel];

    // The keychain access group attribute determines if this item can be shared
    // amongst multiple apps whose code signing entitlements contain the same keychain access group.
    NSString *accessGroup = [NSString stringWithUTF8String:kKeyChainUDIDAccessGroup];

    if (accessGroup != nil) {
#if TARGET_IPHONE_SIMULATOR
        // Ignore the access group if running on the iPhone simulator.
        //
        // Apps that are built for the simulator aren't signed, so there's no keychain access group
        // for the simulator to check. This means that all apps can see all keychain items when run
        // on the simulator.
        //
        // If a SecItem contains an access group attribute, SecItemAdd and SecItemUpdate on the
        // simulator will return -25243 (errSecNoAccessForItem).
#else
        [dictForAdd setObject:accessGroup forKey:(id)kSecAttrAccessGroup];
#endif
    }

    const char *udidStr = [udid UTF8String];
    NSData *keyChainItemValue = [NSData dataWithBytes:udidStr length:strlen(udidStr)];
    [dictForAdd setValue:keyChainItemValue forKey:(id)kSecValueData];

    OSStatus writeErr = noErr;
    if ([ToolKit getUDIDFromKeyChain]) {        // there is item in keychain
        [ToolKit updateUDIDInKeyChain:udid];
        [dictForAdd release];
        return YES;
    } else {          // add item to keychain
        writeErr = SecItemAdd((CFDictionaryRef)dictForAdd, NULL);
        if (writeErr != errSecSuccess) {
            NSLog(@"Add KeyChain Item Error!!! Error Code:%d", (int)writeErr);

            [dictForAdd release];
            return NO;
        } else {
            NSLog(@"Add KeyChain Item Success!!!");
            [dictForAdd release];
            return YES;
        }
    }

    //    [dictForAdd release];
    return NO;
}

+ (BOOL)removeUDIDFromKeyChain {

    NSMutableDictionary *dictToDelete = [[NSMutableDictionary alloc] init];

    [dictToDelete setValue:(id)kSecClassGenericPassword forKey:(id)kSecClass];

    NSData *keyChainItemID = [NSData dataWithBytes:kKeychainUDIDItemIdentifier length:strlen(kKeychainUDIDItemIdentifier)];
    [dictToDelete setValue:keyChainItemID forKey:(id)kSecAttrGeneric];

    OSStatus deleteErr = noErr;
    deleteErr = SecItemDelete((CFDictionaryRef)dictToDelete);

    if (deleteErr != errSecSuccess) {
        NSLog(@"delete UUID from KeyChain Error!!! Error code:%d", (int)deleteErr);
        [dictToDelete release];
        return NO;
    } else {
        NSLog(@"delete success!!!");
    }

    [dictToDelete release];
    return YES;
}

+ (BOOL)updateUDIDInKeyChain:(NSString*)newUDID {

    NSMutableDictionary *dictForQuery = [[NSMutableDictionary alloc] init];

    [dictForQuery setValue:(id)kSecClassGenericPassword forKey:(id)kSecClass];

    NSData *keychainItemID = [NSData dataWithBytes:kKeychainUDIDItemIdentifier length:strlen(kKeychainUDIDItemIdentifier)];
    [dictForQuery setValue:keychainItemID forKey:(id)kSecAttrGeneric];
    [dictForQuery setValue:(id)kCFBooleanTrue forKey:(id)kSecMatchCaseInsensitive];
    [dictForQuery setValue:(id)kSecMatchLimitOne forKey:(id)kSecMatchLimit];
    [dictForQuery setValue:(id)kCFBooleanTrue forKey:(id)kSecReturnAttributes];

    NSDictionary *queryResult = nil;
    SecItemCopyMatching((CFDictionaryRef)dictForQuery, (CFTypeRef*)&queryResult);

    if (queryResult) {

        NSMutableDictionary *dictForUpdate = [[NSMutableDictionary alloc] init];
        [dictForUpdate setValue:[NSString stringWithUTF8String:kKeychainUDIDItemIdentifier] forKey:(id)kSecAttrDescription];
        [dictForUpdate setValue:keychainItemID forKey:(id)kSecAttrGeneric];

        const char *udidStr = [newUDID UTF8String];
        NSData *keyChainItemValue = [NSData dataWithBytes:udidStr length:strlen(udidStr)];
        [dictForUpdate setValue:keyChainItemValue forKey:(id)kSecValueData];

        OSStatus updateErr = noErr;

        // First we need the attributes from the Keychain.
        NSMutableDictionary *updateItem = [NSMutableDictionary dictionaryWithDictionary:queryResult];
        [queryResult release];

        // Second we need to add the appropriate search key/values.
        // set kSecClass is Very important
        [updateItem setObject:(id)kSecClassGenericPassword forKey:(id)kSecClass];

        updateErr = SecItemUpdate((CFDictionaryRef)updateItem, (CFDictionaryRef)dictForUpdate);

        if (updateErr != errSecSuccess) {
            NSLog(@"Update KeyChain Item Error!!! Error Code:%d", (int)updateErr);

            [dictForQuery release];
            [dictForUpdate release];
            return NO;
        } else {
            NSLog(@"Update KeyChain Item Success!!!");
            [dictForQuery release];
            [dictForUpdate release];
            return YES;
        }
    }

    [dictForQuery release];
    return NO;
}

@end
```