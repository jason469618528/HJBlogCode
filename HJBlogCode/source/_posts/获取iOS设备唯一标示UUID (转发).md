---
title: 获取iOS设备唯一标示UUID (转发来源简书)
date: 2017-04-21 09:30:00
tags:
---

在开发过程中，我们经常会被要求获取每个设备的唯一标示，以便后台做相应的处理。我们来看看有哪些方法来获取设备的唯一标示，然后再分析下这些方法的利弊。

具体可以分为如下几种：

1. UDID
2. IDFA
3. IDFV
4. MAC
5. keychain
- - - -
下面我们来具体分析下每种获取方法的利弊
### 1、UDID

什么是UDID

UDID 「Unique Device Identifier Description」是由子母和数字组成的40个字符串的序号，用来区别每一个唯一的iOS设备，包括 iPhones, iPads, 以及 iPod touches，这些编码看起来是随机的，实际上是跟硬件设备特点相联系的，另外你可以到iTunes，pp助手或itools等软件查看你的udid（设备标识）

UDID是用来干什么的？

UDID可以关联其它各种数据到相关设备上。例如，连接到开发者账号，可以允许在发布前让设备安装或测试应用；也可以让开发者获得iOS测试版进行体验。苹果用UDID连接到苹果的ID，这些设备可以自动下载和安装从App Store购买的应用、保存从iTunes购买的音乐、帮助苹果发送推送通知、即时消息。 在iOS 应用早期，UDID被第三方应用开发者和网络广告商用来收集用户数据，可以用来关联地址、记录应用使用习惯……以便推送精准广告。

为什么苹果反对开发人员使用UDID？

iOS 2.0版本以后UIDevice提供一个获取设备唯一标识符的方法uniqueIdentifier，通过该方法我们可以获取设备的序列号，这个也是目前为止唯一可以确认唯一的标示符。 许多开发者把UDID跟用户的真实姓名、密码、住址、其它数据关联起来；网络窥探者会从多个应用收集这些数据，然后顺藤摸瓜得到这个人的许多隐私数据。同时大部分应用确实在频繁传输UDID和私人信息。 为了避免集体诉讼，苹果最终决定在iOS 5 的时候，将这一惯例废除，开发者被引导生成一个唯一的标识符，只能检测应用程序，其他的信息不提供。现在应用试图获取UDID已被禁止且不允许上架。

所以这个方法作废

### 2、IDFA

全名：advertisingIdentifier

获取代码：
`  #import <AdSupport/AdSupport.h>
  NSString *adId = [[[ASIdentifierManager sharedManager] advertisingIdentifier] UUIDString]; `
来源：iOS6.0及以后

说明：直译就是广告id， 在同一个设备上的所有App都会取到相同的值，是苹果专门给各广告提供商用来追踪用户而设的，用户可以在 设置|隐私|广告追踪 里重置此id的值，或限制此id的使用，故此id有可能会取不到值，但好在Apple默认是允许追踪的，而且一般用户都不知道有这么个设置，所以基本上用来监测推广效果，是戳戳有余了。

注意：由于idfa会出现取不到的情况，故绝不可以作为业务分析的主id，来识别用户。

### 3、IDFV

全名：identifierForVendor
获取代码：
`NSString *idfv = [[[UIDevice currentDevice] identifierForVendor] UUIDString];
`
来源：iOS6.0及以后

说明：顾名思义，是给Vendor标识用户用的，每个设备在所属同一个Vender的应用里，都有相同的值。其中的Vender是指应用提供商，但准确点说，是通过BundleID的反转的前两部分进行匹配，如果相同就是同一个Vender，例如对于com.taobao.app1, com.taobao.app2 这两个BundleID来说，就属于同一个Vender，共享同一个idfv的值。和idfa不同的是，idfv的值是一定能取到的，所以非常适合于作为内部用户行为分析的主id，来标识用户，替代OpenUDID。

注意：如果用户将属于此Vender的所有App卸载，则idfv的值会被重置，即再重装此Vender的App，idfv的值和之前不同。

### 4、MAC地址

使用WiFi的mac地址来取代已经废弃了的uniqueIdentifier方法。具体可见:
http://stackoverflow.com/questions/677530/how-can-i-programmatically-get-the-mac-address-of-an-iphone

然而在iOS 7中苹果再一次无情的封杀mac地址，使用之前的方法获取到的mac地址全部都变成了02:00:00:00:00:00。

### 5、Keychain

![](/img/277755-807a54b6dcbeb347.png)

我们可以获取到UUID，然后把UUID保存到KeyChain里面。

这样以后即使APP删了再装回来，也可以从KeyChain中读取回来。使用group还可以可以保证同一个开发商的所有程序针对同一台设备能够获取到相同的不变的UDID。

但是刷机或重装系统后uuid还是会改变。

把下面两个类文件放到你的项目中
```
KeychainItemWrapper.h文件
********************************

#import <UIKit/UIKit.h>

@interface KeychainItemWrapper : NSObject
{
    NSMutableDictionary *keychainItemData;        // The actual keychain item data backing store.
    NSMutableDictionary *genericPasswordQuery;    // A placeholder for the generic keychain item query used to locate the item.
}

@property (nonatomic, retain) NSMutableDictionary *keychainItemData;
@property (nonatomic, retain) NSMutableDictionary *genericPasswordQuery;

// Designated initializer.
- (id)initWithAccount:(NSString *)account service:(NSString *)service accessGroup:(NSString *) accessGroup;

- (id)initWithIdentifier: (NSString *)identifier accessGroup:(NSString *) accessGroup;
- (void)setObject:(id)inObject forKey:(id)key;
- (id)objectForKey:(id)key;

// Initializes and resets the default generic keychain item data.
- (void)resetKeychainItem;

@end
```

```
KeychainItemWrapper.h文件
********************************

#import "KeychainItemWrapper.h"
#import <Security/Security.h>

/*

These are the default constants and their respective types,
available for the kSecClassGenericPassword Keychain Item class:

kSecAttrAccessGroup            -        CFStringRef
kSecAttrCreationDate        -        CFDateRef
kSecAttrModificationDate    -        CFDateRef
kSecAttrDescription            -        CFStringRef
kSecAttrComment                -        CFStringRef
kSecAttrCreator                -        CFNumberRef
kSecAttrType                -        CFNumberRef
kSecAttrLabel                -        CFStringRef
kSecAttrIsInvisible            -        CFBooleanRef
kSecAttrIsNegative            -        CFBooleanRef
kSecAttrAccount                -        CFStringRef
kSecAttrService                -        CFStringRef
kSecAttrGeneric                -        CFDataRef

See the header file Security/SecItem.h for more details.

*/

@interface KeychainItemWrapper (PrivateMethods)
/*
The decision behind the following two methods (secItemFormatToDictionary and dictionaryToSecItemFormat) was
to encapsulate the transition between what the detail view controller was expecting (NSString *) and what the
Keychain API expects as a validly constructed container class.
*/
- (NSMutableDictionary *)secItemFormatToDictionary:(NSDictionary *)dictionaryToConvert;
- (NSMutableDictionary *)dictionaryToSecItemFormat:(NSDictionary *)dictionaryToConvert;

// Updates the item in the keychain, or adds it if it doesn't exist.
- (void)writeToKeychain;

@end

@implementation KeychainItemWrapper

@synthesize keychainItemData, genericPasswordQuery;

- (id)initWithAccount:(NSString *)account service:(NSString *)service accessGroup:(NSString *) accessGroup;
{
    if (self = [super init])
    {
        NSAssert(account != nil || service != nil, @"Both account and service are nil.  Must specifiy at least one.");
        // Begin Keychain search setup. The genericPasswordQuery the attributes kSecAttrAccount and
        // kSecAttrService are used as unique identifiers differentiating keychain items from one another
        genericPasswordQuery = [[NSMutableDictionary alloc] init];

        [genericPasswordQuery setObject:(id)kSecClassGenericPassword forKey:(id)kSecClass];

        [genericPasswordQuery setObject:account forKey:(id)kSecAttrAccount];
        [genericPasswordQuery setObject:service forKey:(id)kSecAttrService];

        // The keychain access group attribute determines if this item can be shared
        // amongst multiple apps whose code signing entitlements contain the same keychain access group.
        if (accessGroup != nil)
        {
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
            [genericPasswordQuery setObject:accessGroup forKey:(id)kSecAttrAccessGroup];
#endif
        }

        // Use the proper search constants, return only the attributes of the first match.
        [genericPasswordQuery setObject:(id)kSecMatchLimitOne forKey:(id)kSecMatchLimit];
        [genericPasswordQuery setObject:(id)kCFBooleanTrue forKey:(id)kSecReturnAttributes];

        NSDictionary *tempQuery = [NSDictionary dictionaryWithDictionary:genericPasswordQuery];

        NSMutableDictionary *outDictionary = nil;

        if (! SecItemCopyMatching((CFDictionaryRef)tempQuery, (CFTypeRef *)&outDictionary) == noErr)
        {
            // Stick these default values into keychain item if nothing found.
            [self resetKeychainItem];

            //Adding the account and service identifiers to the keychain
            [keychainItemData setObject:account forKey:(id)kSecAttrAccount];
            [keychainItemData setObject:service forKey:(id)kSecAttrService];

            if (accessGroup != nil)
            {
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
                [keychainItemData setObject:accessGroup forKey:(id)kSecAttrAccessGroup];
#endif
            }
        }
        else
        {
            // load the saved data from Keychain.
            self.keychainItemData = [self secItemFormatToDictionary:outDictionary];
        }

        [outDictionary release];
    }

    return self;
}

- (id)initWithIdentifier: (NSString *)identifier accessGroup:(NSString *) accessGroup;
{
    if (self = [super init])
    {
        // Begin Keychain search setup. The genericPasswordQuery leverages the special user
        // defined attribute kSecAttrGeneric to distinguish itself between other generic Keychain
        // items which may be included by the same application.
        genericPasswordQuery = [[NSMutableDictionary alloc] init];

        [genericPasswordQuery setObject:(id)kSecClassGenericPassword forKey:(id)kSecClass];
        [genericPasswordQuery setObject:identifier forKey:(id)kSecAttrGeneric];

        // The keychain access group attribute determines if this item can be shared
        // amongst multiple apps whose code signing entitlements contain the same keychain access group.
        if (accessGroup != nil)
        {
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
            [genericPasswordQuery setObject:accessGroup forKey:(id)kSecAttrAccessGroup];
#endif
        }

        // Use the proper search constants, return only the attributes of the first match.
        [genericPasswordQuery setObject:(id)kSecMatchLimitOne forKey:(id)kSecMatchLimit];
        [genericPasswordQuery setObject:(id)kCFBooleanTrue forKey:(id)kSecReturnAttributes];

        NSDictionary *tempQuery = [NSDictionary dictionaryWithDictionary:genericPasswordQuery];

        NSMutableDictionary *outDictionary = nil;

        if (! SecItemCopyMatching((CFDictionaryRef)tempQuery, (CFTypeRef *)&outDictionary) == noErr)
        {
            // Stick these default values into keychain item if nothing found.
            [self resetKeychainItem];

            // Add the generic attribute and the keychain access group.
            [keychainItemData setObject:identifier forKey:(id)kSecAttrGeneric];
            if (accessGroup != nil)
            {
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
                [keychainItemData setObject:accessGroup forKey:(id)kSecAttrAccessGroup];
#endif
            }
        }
        else
        {
            // load the saved data from Keychain.
            self.keychainItemData = [self secItemFormatToDictionary:outDictionary];
        }

        [outDictionary release];
    }

    return self;
}

- (void)dealloc
{
    [keychainItemData release];
    [genericPasswordQuery release];

    [super dealloc];
}

- (void)setObject:(id)inObject forKey:(id)key 
{
    if (inObject == nil) return;
    id currentObject = [keychainItemData objectForKey:key];
    if (![currentObject isEqual:inObject])
    {
        [keychainItemData setObject:inObject forKey:key];
        [self writeToKeychain];
    }
}

- (id)objectForKey:(id)key
{
    return [keychainItemData objectForKey:key];
}

- (void)resetKeychainItem
{
    OSStatus junk = noErr;
    if (!keychainItemData) 
    {
        self.keychainItemData = [[NSMutableDictionary alloc] init];
    }
    else if (keychainItemData)
    {
        NSMutableDictionary *tempDictionary = [self dictionaryToSecItemFormat:keychainItemData];
        junk = SecItemDelete((CFDictionaryRef)tempDictionary);
        NSAssert( junk == noErr || junk == errSecItemNotFound, @"Problem deleting current dictionary." );
    }

    // Default attributes for keychain item.
    [keychainItemData setObject:@"" forKey:(id)kSecAttrAccount];
    [keychainItemData setObject:@"" forKey:(id)kSecAttrLabel];
    [keychainItemData setObject:@"" forKey:(id)kSecAttrDescription];

    // Default data for keychain item.
    [keychainItemData setObject:@"" forKey:(id)kSecValueData];
}

- (NSMutableDictionary *)dictionaryToSecItemFormat:(NSDictionary *)dictionaryToConvert
{
    // The assumption is that this method will be called with a properly populated dictionary
    // containing all the right key/value pairs for a SecItem.

    // Create a dictionary to return populated with the attributes and data.
    NSMutableDictionary *returnDictionary = [NSMutableDictionary dictionaryWithDictionary:dictionaryToConvert];

    // Add the Generic Password keychain item class attribute.
    [returnDictionary setObject:(id)kSecClassGenericPassword forKey:(id)kSecClass];

    // Convert the NSString to NSData to meet the requirements for the value type kSecValueData.
    // This is where to store sensitive data that should be encrypted.
    NSString *passwordString = [dictionaryToConvert objectForKey:(id)kSecValueData];
    [returnDictionary setObject:[passwordString dataUsingEncoding:NSUTF8StringEncoding] forKey:(id)kSecValueData];

    return returnDictionary;
}

- (NSMutableDictionary *)secItemFormatToDictionary:(NSDictionary *)dictionaryToConvert
{
    // The assumption is that this method will be called with a properly populated dictionary
    // containing all the right key/value pairs for the UI element.

    // Create a dictionary to return populated with the attributes and data.
    NSMutableDictionary *returnDictionary = [NSMutableDictionary dictionaryWithDictionary:dictionaryToConvert];

    // Add the proper search key and class attribute.
    [returnDictionary setObject:(id)kCFBooleanTrue forKey:(id)kSecReturnData];
    [returnDictionary setObject:(id)kSecClassGenericPassword forKey:(id)kSecClass];

    // Acquire the password data from the attributes.
    NSData *passwordData = NULL;
    if (SecItemCopyMatching((CFDictionaryRef)returnDictionary, (CFTypeRef *)&passwordData) == noErr)
    {
        // Remove the search, class, and identifier key/value, we don't need them anymore.
        [returnDictionary removeObjectForKey:(id)kSecReturnData];

        // Add the password to the dictionary, converting from NSData to NSString.
        NSString *password = [[[NSString alloc] initWithBytes:[passwordData bytes] length:[passwordData length] 
                                                     encoding:NSUTF8StringEncoding] autorelease];
        [returnDictionary setObject:password forKey:(id)kSecValueData];
    }
    else
    {
        // Don't do anything if nothing is found.
        NSAssert(NO, @"Serious error, no matching item found in the keychain.\n");
    }

    [passwordData release];

    return returnDictionary;
}

- (void)writeToKeychain
{
    NSDictionary *attributes = NULL;
    NSMutableDictionary *updateItem = NULL;
    OSStatus result;

    if (SecItemCopyMatching((CFDictionaryRef)genericPasswordQuery, (CFTypeRef *)&attributes) == noErr)
    {
        // First we need the attributes from the Keychain.
        updateItem = [NSMutableDictionary dictionaryWithDictionary:attributes];
        // Second we need to add the appropriate search key/values.
        [updateItem setObject:[genericPasswordQuery objectForKey:(id)kSecClass] forKey:(id)kSecClass];

        // Lastly, we need to set up the updated attribute list being careful to remove the class.
        NSMutableDictionary *tempCheck = [self dictionaryToSecItemFormat:keychainItemData];
        [tempCheck removeObjectForKey:(id)kSecClass];

#if TARGET_IPHONE_SIMULATOR
        // Remove the access group if running on the iPhone simulator.
        // 
        // Apps that are built for the simulator aren't signed, so there's no keychain access group
        // for the simulator to check. This means that all apps can see all keychain items when run
        // on the simulator.
        //
        // If a SecItem contains an access group attribute, SecItemAdd and SecItemUpdate on the
        // simulator will return -25243 (errSecNoAccessForItem).
        //
        // The access group attribute will be included in items returned by SecItemCopyMatching,
        // which is why we need to remove it before updating the item.
        [tempCheck removeObjectForKey:(id)kSecAttrAccessGroup];
#endif

        // An implicit assumption is that you can only update a single item at a time.

        result = SecItemUpdate((CFDictionaryRef)updateItem, (CFDictionaryRef)tempCheck);
        NSAssert( result == noErr, @"Couldn't update the Keychain Item." );
    }
    else
    {
        // No previous item found; add the new one.
        result = SecItemAdd((CFDictionaryRef)[self dictionaryToSecItemFormat:keychainItemData], NULL);
        NSAssert( result == noErr, @"Couldn't add the Keychain Item." );
    }
}

@end
```
我们在写一个工具类用来保存UUID到keychain和从keychain中读取UUID.

**实现代码**
```
#pragma mark - 保存和读取UUID
+(void)saveUUIDToKeyChain{
    KeychainItemWrapper *keychainItem = [[KeychainItemWrapper alloc] initWithAccount:@"Identfier" service:@"AppName" accessGroup:nil];
    NSString *string = [keychainItem objectForKey: (__bridge id)kSecAttrGeneric];
    if([string isEqualToString:@""] || !string){
        [keychainItem setObject:[self getUUIDString] forKey:(__bridge id)kSecAttrGeneric];
    }
}

+(NSString *)readUUIDFromKeyChain{
    KeychainItemWrapper *keychainItemm = [[KeychainItemWrapper alloc] initWithAccount:@"Identfier" service:@"AppName" accessGroup:nil];
    NSString *UUID = [keychainItemm objectForKey: (__bridge id)kSecAttrGeneric];
    return UUID;
}

+ (NSString *)getUUIDString
{
    CFUUIDRef uuidRef = CFUUIDCreate(kCFAllocatorDefault);
    CFStringRef strRef = CFUUIDCreateString(kCFAllocatorDefault , uuidRef);
    NSString *uuidString = [(__bridge NSString*)strRef stringByReplacingOccurrencesOfString:@"-" withString:@""];
    CFRelease(strRef);
    CFRelease(uuidRef);
    return uuidString;
}
```
写入UUID到keychain

我们最好在程序启动之后把UUID写入到keychain，代码如下：
```
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    [AppUtils saveUUIDToKeyChain];
}
```
读取UUID

在需要读取的地方直接调用AppUtils的类方法readUUIDFromKeyChain即可。
- - - -
1.让同一开发商的所有APP在同一台设备上获取到UUID相同

在每个APP的项目里面做如下设置

1.1、设置accessgroup
```
keychainItemWrapper *keychainItem = [[KeychainItemWrapper alloc] initWithAccount:@"Identfier" service:@"AppName" accessGroup:@"YOUR_BUNDLE_SEED.com.yourcompany.userinfo"];

```
1.2、创建plist文件

然后在项目相同的目录下创建KeychainAccessGroups.plist文件。

该文件的结构是一个字典，其中中最顶层的节点必须是一个键为“keychain-access-groups”的Array，并且该Array中每一项都是一个描述分组的NSString。YOUR_BUNDLE_SEED.com.yourcompany.userinfo就是要设置的组名。

如图:
![](/img/277755-72008ba008a56557.png)
1.3、 设置code signing

接着在Target--->Build Settings---->Code Signing栏下的Code Signing Entitlements右侧添加KeychainAccessGroups.plist

如图：
![](/img/277755-e736892057d71333.png)

这样就可以保证每个app都是从keychain中读取出来同一个UUID
