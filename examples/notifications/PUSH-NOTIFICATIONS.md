# Push Notifications

## Overview

Comprehensive push notification implementation examples for all four major stacks: TALL Stack (Laravel/PHP), Django/Wagtail (Python), React/Next.js (web push), and React Native (mobile). Each implementation includes permission handling, token registration, notification sending, and delivery configuration using industry-standard services like Firebase Cloud Messaging.

## Metadata

| Property            | Value                       |
| ------------------- | --------------------------- |
| **Example Version** | 2.0.0                       |
| **Last Updated**    | 2025-12                     |
| **Laravel**         | 12.x                        |
| **PHP**             | 8.4                         |
| **Django**          | 6.x                         |
| **Python**          | 3.14                        |
| **Next.js**         | 16.x                        |
| **React Native**    | 0.83.x                      |
| **Expo**            | 52.x                        |
| **Stacks**          | TALL, Django, React, Mobile |

---

## Table of Contents

- [Overview](#overview)
- [Metadata](#metadata)
- [Table of Contents](#table-of-contents)
- [Directory Structure](#directory-structure)
  - [Laravel Directory Structure](#laravel-directory-structure)
  - [Django Directory Structure](#django-directory-structure)
  - [Next.js Directory Structure](#nextjs-directory-structure)
  - [React Native Directory Structure](#react-native-directory-structure)
- [Push Configuration - React Native](#push-configuration---react-native)
  - [src/notifications/config/pushConfig.ts](#srcnotificationsconfigpushconfigts)
- [Notification Handler - React Native](#notification-handler---react-native)
  - [src/notifications/handlers/notificationHandler.ts](#srcnotificationshandlersnotificationhandlerts)


## Directory Structure

### Laravel Directory Structure

```
app/
├── Services/
│   └── PushNotificationService.php      # FCM service integration
├── Notifications/
│   ├── OrderStatusNotification.php      # Order update notification
│   ├── WelcomeNotification.php          # Welcome notification
│   └── MessageNotification.php          # Message notification
└── Http/
    └── Controllers/
        └── NotificationController.php   # Notification endpoints
config/
└── services.php                          # FCM credentials
```

### Django Directory Structure

```
notifications/
├── services/
│   └── push_service.py                   # FCM service integration
├── backends/
│   └── fcm_backend.py                    # Django push notifications backend
├── templates/
│   ├── order_notification.py             # Order notification template
│   ├── welcome_notification.py           # Welcome notification template
│   └── message_notification.py           # Message notification template
└── views/
    └── notification_views.py             # API endpoints
settings/
└── base.py                               # Push notifications configuration
```

### Next.js Directory Structure

```
public/
└── service-worker.js                     # Push notification service worker
app/
├── api/
│   └── notifications/
│       ├── subscribe/
│       │   └── route.ts                  # Subscribe endpoint
│       └── send/
│           └── route.ts                  # Send notification endpoint
└── components/
    └── PushNotificationManager.tsx       # Client-side push manager
lib/
└── push-notifications.ts                 # Push notification utilities
```

### React Native Directory Structure

```
src/notifications/
├── config/
│   └── pushConfig.ts                     # Push notification configuration
├── handlers/
│   ├── notificationHandler.ts            # Background/foreground handlers
│   └── deepLinkHandler.ts                # Deep link routing from notifications
├── services/
│   └── notificationService.ts            # Send/receive push notifications
└── templates/
    ├── welcomeNotification.ts            # Welcome push template
    ├── orderNotification.ts              # Order update push template
    └── messageNotification.ts            # Message push template
```

---

## Push Configuration - React Native

### src/notifications/config/pushConfig.ts

```typescript
/**
 * pushConfig.ts
 *
 * Push notification configuration for React Native with Expo.
 * Handles notification permissions and token registration.
 */

import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { Platform } from 'react-native';

/**
 * Configures notification handlers for foreground notifications.
 *
 * Sets up how notifications appear when the app is active.
 * By default, shows alerts, plays sounds, and sets badge counts.
 */
export function configurePushNotifications(): void {
  Notifications.setNotificationHandler({
    handleNotification: async () => ({
      shouldShowAlert: true,
      shouldPlaySound: true,
      shouldSetBadge: true,
    }),
  });
}

/**
 * Registers for push notifications and returns the Expo push token.
 *
 * Checks if running on a physical device (required for push notifications),
 * requests permission if not already granted, and configures the Android
 * notification channel.
 *
 * @returns Promise resolving to the Expo push token or null if registration failed
 */
export async function registerForPushNotifications(): Promise<string | null> {
  if (!Device.isDevice) {
    console.warn('Push notifications require a physical device');
    return null;
  }

  const { status: existingStatus } = await Notifications.getPermissionsAsync();
  let finalStatus = existingStatus;

  if (existingStatus !== 'granted') {
    const { status } = await Notifications.requestPermissionsAsync();
    finalStatus = status;
  }

  if (finalStatus !== 'granted') {
    console.warn('Push notification permission denied');
    return null;
  }

  // Configure Android notification channel
  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'Default',
      importance: Notifications.AndroidImportance.MAX,
      vibrationPattern: [0, 250, 250, 250],
      lightColor: '#1a73e8',
    });
  }

  const token = await Notifications.getExpoPushTokenAsync();
  return token.data;
}

/**
 * Clears all notification badges from the app icon.
 *
 * Should be called when the user opens the notification centre
 * or views their notifications in-app.
 */
export async function clearBadgeCount(): Promise<void> {
  await Notifications.setBadgeCountAsync(0);
}

/**
 * Gets the current notification permissions status.
 *
 * @returns Promise resolving to the permission status
 */
export async function getNotificationPermissionStatus(): Promise<Notifications.PermissionStatus> {
  const { status } = await Notifications.getPermissionsAsync();
  return status;
}
```

---

## Notification Handler - React Native

### src/notifications/handlers/notificationHandler.ts

```typescript
/**
 * notificationHandler.ts
 *
 * Handles incoming push notifications in foreground and background.
 * Routes notification actions to appropriate screens via deep linking.
 */

import * as Notifications from 'expo-notifications';
import { router } from 'expo-router';
import { useEffect, useRef } from 'react';

interface NotificationData {
  type?: string;
  orderId?: string;
  userId?: string;
  screen?: string;
  params?: Record<string, string>;
}

/**
 * Custom hook to handle push notification events.
 *
 * Sets up listeners for incoming notifications (foreground) and
 * notification responses (when user taps). Cleans up listeners
 * on component unmount.
 */
export function useNotificationHandler(): void {
  const notificationListener = useRef<Notifications.Subscription>();
  const responseListener = useRef<Notifications.Subscription>();

  useEffect(() => {
    // Listen for incoming notifications (foreground)
    notificationListener.current = Notifications.addNotificationReceivedListener(
      (notification) => {
        handleIncomingNotification(notification);
      }
    );

    // Listen for notification taps
    responseListener.current = Notifications.addNotificationResponseReceivedListener(
      (response) => {
        handleNotificationResponse(response);
      }
    );

    return () => {
      if (notificationListener.current) {
        Notifications.removeNotificationSubscription(notificationListener.current);
      }
      if (responseListener.current) {
        Notifications.removeNotificationSubscription(responseListener.current);
      }
    };
  }, []);
}

/**
 * Handles incoming notifications while app is in foreground.
 *
 * Logs the notification for debugging and can be extended to
 * show in-app toasts or update local state.
 *
 * @param notification - The received notification object
 */
function handleIncomingNotification(
  notification: Notifications.Notification
): void {
  const data = notification.request.content.data as NotificationData;
  console.log('Notification received:', data);

  // Optionally show in-app toast or update state
}

/**
 * Handles user interaction with a notification.
 *
 * Routes the user to the appropriate screen based on the
 * notification data. Supports order details, user profiles,
 * and generic screen navigation.
 *
 * @param response - The notification response containing user action
 */
function handleNotificationResponse(
  response: Notifications.NotificationResponse
): void {
  const data = response.notification.request.content.data as NotificationData;

  switch (data.type) {
    case 'order_update':
      if (data.orderId) {
        router.push(`/orders/${data.orderId}`);
      }
      break;

    case 'new_message':
      router.push('/messages');
      break;

    case 'account_alert':
      router.push('/settings/security');
      break;

    default:
      // Use generic screen navigation if provided
      if (data.screen) {
        router.push({
          pathname: data.screen,
          params: data.params || {},
        });
      }
      break;
  }
}

/**
 * Schedules a local notification for testing or reminders.
 *
 * @param title - The notification title
 * @param body - The notification body text
 * @param data - Optional data payload for handling taps
 * @param seconds - Delay in seconds before showing (default: 5)
 */
export async function scheduleLocalNotification(
  title: string,
  body: string,
  data?: NotificationData,
  seconds: number = 5
): Promise<string> {
  return await Notifications.scheduleNotificationAsync({
    content: {
      title,
      body,
      data,
    },
    trigger: {
      seconds,
    },
  });
}
```
