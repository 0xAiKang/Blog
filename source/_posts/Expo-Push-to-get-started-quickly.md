---
title: Expo Push快速上手
date: 2023-04-01 14:24:47
tags: ["PHP"]
categories: ["PHP"]
---

[Expo](https://expo.dev) 是国外的一套工具、库和服务，让你可以通过编写JavaScript 来构建本地的 iOS 和 Android 应用程序。

<!-- more -->

而 Expo Push 是 Expo 的推送服务。

推送逻辑如下：
1. 服务端请求Expo 推送服务
2. Expo 判断客户端是 Android 还是 iOS
3. 最后通过 APNs（Apple Push Notification service） 和 FCM （Firebase Cloud Messaging）发送通知

![](https://docs.expo.dev/static/images/sending-notification.png)

因为使用 Expo Push 时，需要提供 ExponentPushToken，而这个 Token 需要通过设备才能拿到，因此需要在本地起一个服务，最终通过手机或模拟器运行。

## 准备

在正式使用之前，需要先把以下工具安装好：
* Node.js
* Git

后续都是使用 Expo CLI 这个工具来管理服务的，可以通过 npx（一个 Node.js 包运行器）来使用它。无需额外安装。

打开终端，通过 Expo CLI 进行登录：
```bash
$ npx expo login
```

如果没有账号，需要先[注册](https://expo.dev/signup)一个 Expo 账号。

创建一个 Expo App：
```bash
$ npx create-expo-app my-app && cd my-app
```

将`App.js` 替换成以下内容：
```js
import { StatusBar } from 'expo-status-bar';
import { useState, useEffect, useRef } from 'react';
import { StyleSheet, Text, View, Button, Platform } from 'react-native';
import * as Device from 'expo-device';
import * as Notifications from 'expo-notifications';

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: false,
    shouldSetBadge: false,
  }),
});

// Can use this function below OR use Expo's Push Notification Tool from: https://expo.dev/notifications
async function sendPushNotification(expoPushToken) {
  const message = {
    to: expoPushToken,
    sound: 'default',
    title: 'Original Title',
    body: 'And here is the body!',
    data: { someData: 'goes here' },
  };

  await fetch('https://exp.host/--/api/v2/push/send', {
    method: 'POST',
    headers: {
      Accept: 'application/json',
      'Accept-encoding': 'gzip, deflate',
      'Content-Type': 'application/json',
    },
    body: JSON.stringify(message),
  });
}

async function registerForPushNotificationsAsync() {
  let token;
  if (Device.isDevice) {
    const { status: existingStatus } = await Notifications.getPermissionsAsync();
    let finalStatus = existingStatus;
    if (existingStatus !== 'granted') {
      const { status } = await Notifications.requestPermissionsAsync();
      finalStatus = status;
    }
    if (finalStatus !== 'granted') {
      alert('Failed to get push token for push notification!');
      return;
    }
    token = (await Notifications.getExpoPushTokenAsync()).data;
    console.log(token);
  } else {
    alert('Must use physical device for Push Notifications');
  }

  if (Platform.OS === 'android') {
    Notifications.setNotificationChannelAsync('default', {
      name: 'default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: '#FF231F7C',
    });
  }

  return token;
}

export default function App() {
  const [expoPushToken, setExpoPushToken] = useState('');
  const [notification, setNotification] = useState(false);
  const notificationListener = useRef();
  const responseListener = useRef();

  useEffect(() => {
    registerForPushNotificationsAsync().then(token => setExpoPushToken(token));

    notificationListener.current = Notifications.addNotificationReceivedListener(notification => {
      setNotification(notification);
    });

    responseListener.current = Notifications.addNotificationResponseReceivedListener(response => {
      console.log(response);
    });

    return () => {
      Notifications.removeNotificationSubscription(notificationListener.current);
      Notifications.removeNotificationSubscription(responseListener.current);
    };
  }, []);

  return (
    <View style={{ flex: 1, alignItems: 'center', justifyContent: 'space-around' }}>
      <Text>Your expo push token: {expoPushToken}</Text>
      <View style={{ alignItems: 'center', justifyContent: 'center' }}>
        <Text>Title: {notification && notification.request.content.title} </Text>
        <Text>Body: {notification && notification.request.content.body}</Text>
        <Text>Data: {notification && JSON.stringify(notification.request.content.data)}</Text>
      </View>
      <Button
        title="Press to Send Notification"
        onPress={async () => {
          await sendPushNotification(expoPushToken);
        }}
      />
    </View>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#fff',
    alignItems: 'center',
    justifyContent: 'center',
  },
});
```

启动服务：
```bash
npx expo start
```

顺利的话，应该能看到以下输出，此时需要拿出手机，打开应用商店，搜索安装 Expo 这个 App，然后扫猫下面的这个二维码：

![](https://cdn.jsdelivr.net/gh/0xAiKang/CDN/blog/images/20230220143321.png)

接下来，只需要服务端接入 Expo Push 就可以了。Expo 提供了[各种语言](https://docs.expo.dev/push-notifications/sending-notifications/#send-push-notifications-using-a-server)的推送服务。


下面使用 PHP 接入。

## Expo Push

首先安装依赖包：
```bash
$ composer require ctwillie/expo-server-sdk-php
```

调用实例：
```php
public function push(string $title, string $body, array $expoPushToken)
{
   if (empty($expoPushToken)) {
       return false;
   }

   $messages = [
       new ExpoMessage([
           'title' => $title,
           'body'  => $body,
       ]),
   ];

   try {
       return (new Expo)->send($messages)->to($expoPushToken)->push();
   } catch (\ExpoSDK\Exceptions\ExpoException | \ExpoSDK\Exceptions\InvalidTokensException | Exception $e) {
       return false;
   }
}
```

只需要很少的代码，就可以完成推送了，Expo Push 还支持很多[其他参数](https://docs.expo.dev/push-notifications/sending-notifications/#formats)，可以实现更多需求。

## 测试推送
可以直接调用上面的代码，进行推送测试，也可以通过 Expo 提供的[在线工具](https://expo.dev/notifications)进行测试。

## 参考链接
* [expo-server-sdk-php Github](https://github.com/ctwillie/expo-server-sdk-php)
* [安装基础工具](https://docs.expo.dev/get-started/installation/)
* [Create a new app](https://docs.expo.dev/get-started/create-a-new-app/)
* [推送通知设置](https://docs.expo.dev/push-notifications/push-notifications-setup/)
* [使用 Expo Push Api 发送通知](https://docs.expo.dev/push-notifications/sending-notifications/)
* [使用 FCM 和 APNs 发送通知](https://docs.expo.dev/push-notifications/sending-notifications-custom/#how-to-write-fcm-and-apns-servers)