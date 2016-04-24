---
layout: post
title: （转）Android 判断用户2G/3G/4G移动数据网络
date: 2015-2-20
---

在做 Android App 的时候，为了给用户省流量，为了不激起用户的愤怒，为了更好的用户体验，是需(要根据用户当前网络情况来做一些调整的，也可以在 App 的设置模块里，让用户自己选择，在 2G / 3G / 4G 网络条件下，是否允许请求一些流量比较大的数据。<br />
通过 Android 提供的 TelephonyManager 和 ConnectivityManager 都可以获取到 NetworksInfo 对象，可以通过 getType() 获取类型，判断是 wifi 还是 mobile ，如果是 mobile ，可以通过 NetworksInfo 对象的 getSubType() 和 getSubTypeName() 可以获取到对于的网络类型和名字。<br />
网络类型和名字定义在 TelephonyManager 类里。  <br />

```java
/** Network type is unknown */
public static final int NETWORK_TYPE_UNKNOWN = 0;
/** Current network is GPRS */
public static final int NETWORK_TYPE_GPRS = 1;
/** Current network is EDGE */
public static final int NETWORK_TYPE_EDGE = 2;
/** Current network is UMTS */
public static final int NETWORK_TYPE_UMTS = 3;
/** Current network is CDMA: Either IS95A or IS95B*/
public static final int NETWORK_TYPE_CDMA = 4;
/** Current network is EVDO revision 0*/
public static final int NETWORK_TYPE_EVDO_0 = 5;
/** Current network is EVDO revision A*/
public static final int NETWORK_TYPE_EVDO_A = 6;
/** Current network is 1xRTT*/
public static final int NETWORK_TYPE_1xRTT = 7;
/** Current network is HSDPA */
public static final int NETWORK_TYPE_HSDPA = 8;
/** Current network is HSUPA */
public static final int NETWORK_TYPE_HSUPA = 9;
/** Current network is HSPA */
public static final int NETWORK_TYPE_HSPA = 10;
/** Current network is iDen */
public static final int NETWORK_TYPE_IDEN = 11;
/** Current network is EVDO revision B*/
public static final int NETWORK_TYPE_EVDO_B = 12;
/** Current network is LTE */
public static final int NETWORK_TYPE_LTE = 13;
/** Current network is eHRPD */
public static final int NETWORK_TYPE_EHRPD = 14;
/** Current network is HSPA+ */
public static final int NETWORK_TYPE_HSPAP = 15;
```

<br />看到这个代码和注释，相信没有这方面知识的人很难看懂，都啥玩意？这注释跟没注释有啥区别？！就是让人看着更加闹心而已。所以说，注释对阅读代码的人很重要。当然这些东西可能太专业了，写这些代码的人估计是想写也不知道该怎么了，得写多大一坨啊？！我在最后会贴上一些我整理的资料，可以供大家参考一下，不是很详细，也不专业，就是大概有个印象。<br />
TelephonyManager 还提供了 getNetworkTypeName(int type) 的方法，这个方法可以返回一个字符串，但是信息量不大。<br />
那怎么判断是 2G ， 3G 还是 4G 网络呢？TelephonyManager 还提供了另外一个方法，getNetworkClass(int networkType) ，但这个方法被隐藏掉了，我把代码贴一下。<br /><br />

```java
public static int getNetworkClass(int networkType) {
    switch (networkType) {
        case NETWORK_TYPE_GPRS:
        case NETWORK_TYPE_EDGE:
        case NETWORK_TYPE_CDMA:
        case NETWORK_TYPE_1xRTT:
        case NETWORK_TYPE_IDEN:
    return NETWORK_CLASS_2_G;
        case NETWORK_TYPE_UMTS:
        case NETWORK_TYPE_EVDO_0:
        case NETWORK_TYPE_EVDO_A:
        case NETWORK_TYPE_HSDPA:
        case NETWORK_TYPE_HSUPA:
        case NETWORK_TYPE_HSPA:
        case NETWORK_TYPE_EVDO_B:
        case NETWORK_TYPE_EHRPD:
        case NETWORK_TYPE_HSPAP:
    return NETWORK_CLASS_3_G;
        case NETWORK_TYPE_LTE:
    return NETWORK_CLASS_4_G;
        default:
    return NETWORK_CLASS_UNKNOWN;
    }
}
```
<br />然后下面是这几个常量的值。<br />

```java
/** Unknown network class. {@hide} */
public static final int NETWORK_CLASS_UNKNOWN = 0;
/** Class of broadly defined "2G" networks. {@hide} */
public static final int NETWORK_CLASS_2_G = 1;
/** Class of broadly defined "3G" networks. {@hide} */
public static final int NETWORK_CLASS_3_G = 2;
/** Class of broadly defined "4G" networks. {@hide} */
public static final int NETWORK_CLASS_4_G = 3;
```
<br />不知道为啥要把这些东西给隐藏起来，然道是不靠谱？！还是其他的更好的方式？！不知道，先这样吧，现在通过上面的手段，是可以知道用户用的是什么网络，当然也可以区分出来用户使用的是 2G ， 3G 还是 4G 了。当然，你获取到这些数据后，你也可以推算出用户用的是哪家公司的网络，移动的，联通的，还是电信的，当然，只在中国。而且虚拟运营商开始真正上市后，这个就区分不出来是京东的，还是国美，苏宁的了，但是你可以知道你的手机号用的是联通的网还是移动的网。