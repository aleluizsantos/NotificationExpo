# Notification Expo
<p align="center">
  <img src="./assets/Screenshot1.jpg" width="350" title="Notification Expo">
</p>

O expo-Notifications fornece uma API para buscar tokens de notificação push e para apresentar, agendar, receber e responder a notificações.

    $ expo install expo-notifications 

## Android
Open your app.json and add the following inside of the "expo" field:

    {
    "expo": {
        ...
        "android": {
        ...
        "useNextNotificationsApi": true,
        }
    }
    }

## API

```javascript 
    import * as Notifications from 'expo-notifications';
```

Confira o Lanche abaixo para ver as notificações em ação, mas certifique-se de usar um dispositivo físico! As notificações push não funcionam em simuladores / emuladores.

```javascript
import Constants from "expo-constants";
import * as Notifications from "expo-notifications";
import React, { useState, useEffect } from "react";
import { Text, View, Button, Platform } from "react-native";

Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowAlert: true,
    shouldPlaySound: true,
    shouldSetBadge: false,
  }),
});

export default function App() {
  const [expoPushToken, setExpoPushToken] = useState<string>("");
  const [notification, setNotification] = useState({} as any);

  useEffect(() => {
    registerForPushNotificationsAsync().then((token) =>
      setExpoPushToken(token || "")
    );

    //When app is closed
    const backgroundSubscription = Notifications.addNotificationResponseReceivedListener(
      (response) => {
        console.log("==>", response);
      }
    );
    //When the app is open
    const foregroundSubscription = Notifications.addNotificationReceivedListener(
      (notification) => {
        setNotification(notification);
      }
    );

    return () => {
      backgroundSubscription.remove();
      foregroundSubscription.remove();
    };
  }, []);

  return (
    <View
      style={{
        flex: 1,
        alignItems: "center",
        justifyContent: "space-around",
      }}
    >
      <View
        style={{ alignItems: "center", justifyContent: "center", margin: 20 }}
      >
        <Text>token: {expoPushToken}</Text>
      </View>
      <Button
        title="Click para enviar notificação"
        onPress={async () => {
          await schedulePushNotification();
        }}
      />
    </View>
  );
}

async function schedulePushNotification() {
  await Notifications.scheduleNotificationAsync({
    content: {
      title: "Olá aqui title Notificate! 📬",
      body: "aqui o texto do corpo.",
      data: { data: new Date() },
    },
    trigger: { seconds: 2 },
  });
}

async function registerForPushNotificationsAsync() {
  let token;
  if (Constants.isDevice) {
    const {
      status: existingStatus,
    } = await Notifications.getPermissionsAsync();
    let finalStatus = existingStatus;
    if (existingStatus !== "granted") {
      const { status } = await Notifications.requestPermissionsAsync();
      finalStatus = status;
    }
    if (finalStatus !== "granted") {
      alert("Failed to get push token for push notification!");
      return;
    }
    token = (await Notifications.getExpoPushTokenAsync()).data;
  } else {
    alert("Must use physical device for Push Notifications");
  }

  if (Platform.OS === "android") {
    Notifications.setNotificationChannelAsync("default", {
      name: "default",
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: "#FF231F7C",
    });
  }

  return token;
}
```
