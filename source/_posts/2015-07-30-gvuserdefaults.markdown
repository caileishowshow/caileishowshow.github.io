---
layout: post
title: "GVUserDefaults学习笔记"
date: 2015-07-30 23:21:27 +0800
comments: true
categories: iOS
---
一般来说，对于简单的键值对存储，使用NSUserDefault方法来储存是一个很不错的方法，通过NSUserDefault的setvalue forKey方法可以设置存储变量。
GVUserDefault是一个NSUserDefault的扩展，通过扩展一个实体类的方式来进行键值对的存储，类似于下面的代码
```
// .h
@interface GVUserDefaults (Properties)
@property (nonatomic, weak) NSString *userName;
@property (nonatomic, weak) NSNumber *userId;
@property (nonatomic) NSInteger integerValue;
@property (nonatomic) BOOL boolValue;
@property (nonatomic) float floatValue;
@end

// .m
@implementation GVUserDefaults (Properties)
@dynamic userName;
@dynamic userId;
@dynamic integerValue;
@dynamic boolValue;
@dynamic floatValue;
@end
```
之后直接使用**[GVUserDefaults standardUserDefaults].userName**来代替一般使用的**[[NSUserDefaults standardUserDefaults] objectForKey:@"userName"]**方法，类似如下
```
[GVUserDefaults standardUserDefaults].userName = @"myusername";
```
相对于原来的方法，还是提供了些许的简便，杜绝了因为输入错key导致的一些低级错误。

###源码分析
GVUserDefault 的文件很简单，只有一个.h一个.m文件，其中使用了很多runtime 的相关概念。
而在GVUserDefault 也很简单，只有一个单例方法
```
+ (instancetype)standardUserDefaults;
```
打开.m文件，找到单例的实现方法,也是十分基本的单例实现方法，通过GCD的方式创建单例
```
+ (instancetype)standardUserDefaults {
    static dispatch_once_t pred;
    static GVUserDefaults *sharedInstance = nil;
    dispatch_once(&pred, ^{ sharedInstance = [[self alloc] init]; });
    return sharedInstance;
}
```
再看init方法，首先看一下有没有实现setupDefault方法，该方法使用来设置默认值的，如果实现了这个方法，就设置NSuserdefault的默认值
```
		SEL setupDefaultSEL = NSSelectorFromString([NSString stringWithFormat:@"%@pDefaults", @"setu"]);
        if ([self respondsToSelector:setupDefaultSEL]) {
        	//设置默认值
        }
        [self generateAccessorMethods];
```
最后执行的**- (void)generateAccessorMethods** 是创建构造的重要部分。
首先通过class_copyPropertyList方法得到该类的property 数组，命名为*properties，数组中存放objc_property_t 格式的数据，改结构体在runtime.h 中有说明
```
    unsigned int count = 0;
    objc_property_t *properties = class_copyPropertyList([self class], &count);
```
接下来就是遍历整个数组，取出每一个objc_property_t 结构体。每个结构体中包括两个参数，name和attributes
```
	objc_property_t property = properties[i];
    const char *name = property_getName(property);
    const char *attributes = property_getAttributes(property);
```
name就是这个属性的名称，比如userName、userId等等
attribute也是一个字符串，类似于**T@"NSString",W,D,N**这样，可以认为是一个属性的标志，包括属性类型，以及特定的读写方式等，以后会详细谈到。
在接下来的代码就是根据这个attribute字符串做出一些判断，如果我们的属性指定了getter方法，就是说属性@property 中加入了**getter=Method** 属性，那么在attribute 字符串会有,G+属性名称的字符串，如果判断有此字符串，则用此字符串来构建getter方法，如果没有的话，就用变量名称来构建为getter方法，最后生成方法的SEL
```
        const char *name = property_getName(property);//得到@property 名称
        const char *attributes = property_getAttributes(property);//@property 属性

        char *getter = strstr(attributes, ",G");//判断属性中是否有 getter方法
        if (getter) {
            getter = strdup(getter + 2);//如果有，就用该方法名生成方法
            getter = strsep(&getter, ",");
        } else {
            getter = strdup(name);//如果没有，则用属性名生成方法
        }
        SEL getterSel = sel_registerName(getter);//生成方法的SEL描述
```
接下来是生成setter方法，大体的逻辑和生成getter的基本相同，判断的是",S"字符串，生成的方法名则是setXxx (首字母)，其余基本差不多
接下来将生成的属性名和属性get，set方法SEL描述存放到集合之中。
接下来要做的就是将方法绑定到该class上了
首先生成两个方法指针 getterImp 和 setterImp，接下来通过@property的属性的第二个字符来判断该属性的类型，之后通过不同的类型，选择不同的方法。
在类的上半部分定义了很多的方法，类此这样
```
static bool boolGetter(GVUserDefaults *self, SEL _cmd) {
    NSString *key = [self defaultsKeyForSelector:_cmd];
    return [self.userDefaults boolForKey:key];
}

static void boolSetter(GVUserDefaults *self, SEL _cmd, bool value) {
    NSString *key = [self defaultsKeyForSelector:_cmd];
    [self.userDefaults setBool:value forKey:key];
}

static int integerGetter(GVUserDefaults *self, SEL _cmd) {
    NSString *key = [self defaultsKeyForSelector:_cmd];
    return (int)[self.userDefaults integerForKey:key];
}

static void integerSetter(GVUserDefaults *self, SEL _cmd, int value) {
    NSString *key = [self defaultsKeyForSelector:_cmd];
    [self.userDefaults setInteger:value forKey:key];
}

```
就是根据不同类型选择声称该方法，也是runtime 的一个基本用法，最后再调用**class_addMethod** 方法，来将getter方法和setter 方法写入到类中。
```
	snprintf(types, 4, "%c@:", type);
    class_addMethod([self class], getterSel, getterImp, types);//getter方法写入类
    snprintf(types, 5, "v@:%c", type);
    class_addMethod([self class], setterSel, setterImp, types);//setter方法写入类
```

####总结
总的来说，这个类就是将自定义生成的@property属性创建生成getter 和setter 方法，并在方法中加入NSUserDefault的setValue和getValue 方法，从而达到本地保存的效果