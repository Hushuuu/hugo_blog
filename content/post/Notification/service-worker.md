---
title: "Notification - Service Worker篇"
description: "發送桌面通知"
date: 2021-09-06T16:49:16+08:00
image: 
draft: false
license: 
hidden: false
comments: true
categories:
    - Notification
tags: [
    "Notification",
]
---

## 前言

`Service Worker`目前`Apple`體系似乎還未完全支援`(IOS,Safari...)`   
這次使用`Notification API`  
較舊的瀏覽器可能也無法使用，`Chrome`，`Edge`這些就可以使用了  

## 使用ServiceWorker並訂閱通知

```javascript
function askForNotificationPermission() {
    Notification.requestPermission(function (status) {
        console.log('User Choice', status);
        if (status !== 'granted') {
            console.log('推播允許被拒絕了!');
        } else {
            console.log('推播允許!');
        }
    });
}
//一組web push用的Application Server Public Key
var applicationServerPublicKey = 'BJZStZELls2cT9GaQ29RQlk6wuWGz-RnDFylb2YG77yiEY0DFhyVNG93U-RbUswECsVrt7dfhXLBldmUOGmWqPs';
//註冊service worker js的路徑
var serviceWorker = '../../serviceworker.js';
var isSubscribed = false;

$(document).ready(function () {
    if (typeof applicationServerPublicKey === 'undefined') {
        errorHandler('Vapid public key is undefined.');
        return;
    }
    //初始化sevice worker
    initialiseServiceWorker();
    //判斷browser能不能傳送通知
    Notification.requestPermission().then(function (status) {
        if (status === 'denied') {
            errorHandler('[Notification.requestPermission] Browser denied permissions to notification api.');
        } else if (status === 'granted') {
            console.log('[Notification.requestPermission] Initializing service worker.');   
        }
    });
    //訂閱通知
    subscribe();
});

//初始化並註冊service worker
function initialiseServiceWorker() {
    if ('serviceWorker' in navigator) {
        navigator.serviceWorker.register(serviceWorker).then(handleSWRegistration);
    } else {
        errorHandler('[initialiseServiceWorker] Service workers are not supported in this browser.');
    }
};

//註冊的狀態
function handleSWRegistration(reg) {
    if (reg.installing) {
        console.log('Service worker installing');
    } else if (reg.waiting) {
        console.log('Service worker installed');
    } else if (reg.active) {
        console.log('Service worker active');
    }

    initialiseState(reg);
}

//初始化註冊狀態並檢驗能不能推送通知
function initialiseState(reg) {
    // Are Notifications supported in the service worker?
    if (!(reg.showNotification)) {
        errorHandler('[initialiseState] Notifications aren\'t supported on service workers.');
        return;
    }

    // Check if push messaging is supported
    if (!('PushManager' in window)) {
        errorHandler('[initialiseState] Push messaging isn\'t supported.');
        return;
    }

    // We need the service worker registration to check for a subscription
    navigator.serviceWorker.ready.then(function (reg) {
        // Do we already have a push message subscription?
        reg.pushManager.getSubscription()
            .then(function (subscription) {
                isSubscribed = subscription;
                if (isSubscribed) {
                    console.log('User is already subscribed to push notifications');
                } else {
                    console.log('User is not yet subscribed to push notifications');
                }
            })
            .catch(function (err) {
                console.log('[req.pushManager.getSubscription] Unable to get subscription details.', err);
            });
    });
}

//訂閱通知
function subscribe() {
    navigator.serviceWorker.ready.then(function (reg) {
        var subscribeParams = { userVisibleOnly: true };

        //Setting the public key of our VAPID key pair.
        var applicationServerKey = urlB64ToUint8Array(applicationServerPublicKey);
        subscribeParams.applicationServerKey = applicationServerKey;

        reg.pushManager.subscribe(subscribeParams)
            .then(function (subscription) {
                isSubscribed = true;

                var p256dh = base64Encode(subscription.getKey('p256dh'));
                var auth = base64Encode(subscription.getKey('auth'));
                //console.log(subscription);
                //console.log(subscription.endpoint);
                //console.log(p256dh);
                //console.log(auth);
            })
            .catch(function (e) {
                errorHandler('[subscribe] Unable to subscribe to push', e);
            });
    });
}

//錯誤處理
function errorHandler(message, e) {
    if (typeof e == 'undefined') {
        e = null;
    }
    console.error(message, e);
}

function urlB64ToUint8Array(base64String) {
    var padding = '='.repeat((4 - base64String.length % 4) % 4);
    var base64 = (base64String + padding)
        .replace(/\-/g, '+')
        .replace(/_/g, '/');

    var rawData = window.atob(base64);
    var outputArray = new Uint8Array(rawData.length);

    for (var i = 0; i < rawData.length; ++i) {
        outputArray[i] = rawData.charCodeAt(i);
    }
    return outputArray;
}

function base64Encode(arrayBuffer) {
    return btoa(String.fromCharCode.apply(null, new Uint8Array(arrayBuffer)));
}
```
`serviceworker.js內容`
```javascript
(function () {
    'use strict';

    // Update 'version' if you need to refresh the cache
    var version = 'v1.0::CacheFirstSafe';
    var offlineUrl = ""; // <-- Offline/Index.cshtml
    var urlsToCache = ['/', offlineUrl]; // <-- Add more URLs you would like to cache.

    // Store core files in a cache (including a page to display when offline)
    function updateStaticCache() {
        return caches.open(version)
            .then(function (cache) {
                return cache.addAll(urlsToCache);
            });
    }

    function addToCache(request, response) {
        if (!response.ok && response.type !== 'opaque')
            return;

        var copy = response.clone();
        caches.open(version)
            .then(function (cache) {
                cache.put(request, copy);
            });
    }

    function serveOfflineImage(request) {
        if (request.headers.get('Accept').indexOf('image') !== -1) {
            return new Response('<svg role="img" aria-labelledby="offline-title" viewBox="0 0 400 300" xmlns="http://www.w3.org/2000/svg"><title id="offline-title">Offline</title><g fill="none" fill-rule="evenodd"><path fill="#D8D8D8" d="M0 0h400v300H0z"/><text fill="#9B9B9B" font-family="Helvetica Neue,Arial,Helvetica,sans-serif" font-size="72" font-weight="bold"><tspan x="93" y="172">offline</tspan></text></g></svg>', { headers: { 'Content-Type': 'image/svg+xml' } });
        }
    }

    self.addEventListener('install', function (event) {
        event.waitUntil(updateStaticCache());
    });

    self.addEventListener('activate', function (event) {
        event.waitUntil(
            caches.keys()
                .then(function (keys) {
                    // Remove caches whose name is no longer valid
                    return Promise.all(keys
                        .filter(function (key) {
                            return key.indexOf(version) !== 0;
                        })
                        .map(function (key) {
                            return caches.delete(key);
                        })
                    );
                })
        );
    });

    self.addEventListener('fetch', function (event) {
        var request = event.request;

        // Always fetch non-GET requests from the network
        if (request.method !== 'GET' || request.url.match(/\/browserLink/ig)) {
            event.respondWith(
                fetch(request)
                    .catch(function () {
                        return caches.match(offlineUrl);
                    })
            );
            return;
        }

        // For HTML requests, try the network first, fall back to the cache, finally the offline page
        if (request.headers.get('Accept').indexOf('text/html') !== -1) {
            event.respondWith(
                fetch(request)
                    .then(function (response) {
                        // Stash a copy of this page in the cache
                        addToCache(request, response);
                        return response;
                    })
                    .catch(function () {
                        return caches.match(request)
                            .then(function (response) {
                                return response || caches.match(offlineUrl);
                            });
                    })
            );
            return;
        }

        // cache first for fingerprinted resources
        if (request.url.match(/(\?|&)v=/ig)) {
            event.respondWith(
                caches.match(request)
                    .then(function (response) {
                        return response || fetch(request)
                            .then(function (response) {
                                addToCache(request, response);
                                return response || serveOfflineImage(request);
                            })
                            .catch(function () {
                                return serveOfflineImage(request);
                            });
                    })
            );

            return;
        }

        // network first for non-fingerprinted resources
        event.respondWith(
            fetch(request)
                .then(function (response) {
                    // Stash a copy of this page in the cache
                    addToCache(request, response);
                    return response;
                })
                .catch(function () {
                    return caches.match(request)
                        .then(function (response) {
                            return response || serveOfflineImage(request);
                        })
                        .catch(function () {
                            return serveOfflineImage(request);
                        });
                })
        );
    });
    //監聽PUSH事件
    self.addEventListener('push', function (event) {
        if (!(self.Notification && self.Notification.permission === 'granted')) {
            return;
        }

        var data = {};
        if (event.data) {
            try {
                if (event.data.json()) { data = event.data.json(); }
            }
            catch {
                data.title = "";
                data.message = event.data.text();
            }
        }
        console.log('Notification Received:');
        console.log(data);
        //將接收到的json解析後進行通知
        var title = data.title;
        var message = data.message;
        var icon = data.icon;
        //顯示通知
        event.waitUntil(self.registration.showNotification(title, {
            body: message,
            icon: icon,
            //badge: icon
        }));
    });

})();
```
### 後端推送
可以建立`API`或是`Console`程式來實現推送行為  

```C#
//using WebPush套件
 [HttpPost]
public ActionResult Send(string PushEndpoint,string PushP256DH,string PushAuth,string msg=null)
{
    var payload = "{\"title\":\"this is title\",\"message\":\"this is messaage" + msg + "\"}";
    string payloadstr = JsonConvert.SerializeObject(payload);

    string vapidPublicKey = "BJZStZELls2cT9GaQ29RQlk6wuWGz-RnDFylb2YG77yiEY0DFhyVNG93U-RbUswECsVrt7dfhXLBldmUOGmWqPs";
    string vapidPrivateKey = "f_Rhef2okkI3ZziDLs0jvVGKzVMZHSDWqJrfWJYkbqI";

    var pushSubscription = new PushSubscription(PushEndpoint, PushP256DH, PushAuth);
    var vapidDetails = new VapidDetails("mailto:example@example.com", vapidPublicKey, vapidPrivateKey);

    var webPushClient = new WebPushClient();
    webPushClient.SendNotification(pushSubscription, payload,vapidDetails);

    return Content("");
}
```

## 小結

桌面通知還能再結合`PWA`來讓`Web`更像一個`APP`來使用  
不過不管是`PWA`還是`ServiceWorker`支援度似乎還未很完善尤其是`IOS`環境   

## 參考連結

>* [url1](https://whien.medium.com/%E5%BB%BA%E7%AB%8B-service-worker-web-push-notification-web-notification-%E5%AF%A6%E4%BD%9C%E7%B4%80%E9%8C%84-8a3bb9ff09e8)
>* [url2](https://developers.google.com/web/fundamentals/codelabs/push-notifications?hl=zh-tw)
>* [url3](https://web-push-codelab.glitch.me/)