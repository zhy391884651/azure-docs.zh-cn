---
title: "通知中心本地化的突发新闻教程 - iOS"
description: "了解如何使用 Azure 服务总线通知中心发送本地化的突发新闻通知 (iOS)。"
services: notification-hubs
documentationcenter: ios
author: ysxu
manager: erikre
editor: 
ms.assetid: 484914b5-e081-4a05-a84a-798bbd89d428
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: ios
ms.devlang: objective-c
ms.topic: article
ms.date: 10/03/2016
ms.author: yuaxu
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: fd2b7d9dfd4f432bbcbaa3ed76f8bec0b9677e17


---
# <a name="use-notification-hubs-to-send-localized-breaking-news-to-ios-devices"></a>使用通知中心将本地化的突发新闻发送到 iOS 设备
> [!div class="op_single_selector"]
> * [Windows 应用商店 C#](notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md)
> * [iOS](notification-hubs-ios-xplat-localized-apns-push-notification.md)
> 
> 

## <a name="overview"></a>概述
本主题介绍如何使用 Azure 通知中心的[模板](notification-hubs-templates-cross-platform-push-messages.md)功能广播已按语言和设备本地化的突发新闻通知。 在本教程中，将从在[使用通知中心发送突发新闻]中创建的 iOS 应用开始操作。 完成时，你将可以注册感兴趣的突发新闻类别，指定要接收通知的语言并仅接收采用该语言的这些类别的推送通知。

此方案包含两个部分：

* iOS 应用程序允许客户端设备指定一种语言并订阅不同的突发新闻类别；
* 后端使用 Azure 通知中心的**标记**和**模板**功能广播通知。

## <a name="prerequisites"></a>先决条件
你必须已完成学习[使用通知中心发送突发新闻]教程并具有可用的代码，因为本教程直接围绕该代码展开论述。

可以选用 Visual Studio 2012 或更高版本。

## <a name="template-concepts"></a>模板概念
在[使用通知中心发送突发新闻]中，你构建了一个使用**标记**订阅不同新闻类别通知的应用程序。
但是，很多应用程序针对多个市场，需要本地化。 这意味着通知内容本身必须本地化且传递到正确的设备组。
在本主题中，我们将介绍如何使用通知中心的**模板**功能轻松传递本地化的突发新闻通知。

注意：发送本地化的通知的一种方式是创建每个标签的多个版本。 例如，要支持英语、法语和汉语，我们需要三种不同的标签用于世界新闻：“world_en”、“world_fr”和“world_ch”。 我们然后必须将世界新闻的本地化版本分别发送到这些标签。 在本主题中，我们使用模板来避免增生标签和发送多个消息的要求。

在较高级别上，模板是指定特定设备应如何接收通知的一种方法。 模板通过引用作为你应用程序后端所发消息的一部分的属性，指定确切的负载格式。 在我们的示例中，我们将发送包含所有支持的语言的区域设置未知的消息：

    {
        "News_English": "...",
        "News_French": "...",
        "News_Mandarin": "..."
    }

然后我们将确保设备注册到引用正确属性的模板。 例如，要注册法语新闻的 iOS 应用将注册以下模板：

    {
        aps:{
            alert: "$(News_French)"
        }
    }

模板是很强大的功能，你可以在[模板](notification-hubs-templates-cross-platform-push-messages.md)一文中了解其更多信息。

## <a name="the-app-user-interface"></a>应用程序用户界面
我们现在修改在[使用通知中心发送突发新闻]主题中创建的“突发新闻”应用，以使用模板发送本地化的突发新闻。

在 MainStoryboard_iPhone.storyboard 中，添加使用我们支持的三种语言的分段控件：英语、法语和汉语。

![][13]

然后确保在 ViewController.h 中添加 IBOutlet，如下所示：

![][14]

## <a name="building-the-ios-app"></a>构建 iOS 应用程序
1. 在 Notification.h 中，添加 *retrieveLocale* 方法，然后修改存储区和 subscribe 方法，如下所示：
   
        - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet*) categories completion: (void (^)(NSError* error))completion;
   
        - (void) subscribeWithLocale:(int) locale categories:(NSSet*) categories completion:(void (^)(NSError *))completion;
   
        - (NSSet*) retrieveCategories;
   
        - (int) retrieveLocale;
   
    在 Notification.m 中，通过添加区域设置参数并将它存储在用户默认值中，修改 *storeCategoriesAndSubscribe* 方法：
   
        - (void) storeCategoriesAndSubscribeWithLocale:(int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion {
            NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
            [defaults setValue:[categories allObjects] forKey:@"BreakingNewsCategories"];
            [defaults setInteger:locale forKey:@"BreakingNewsLocale"];
            [defaults synchronize];
   
            [self subscribeWithLocale: locale categories:categories completion:completion];
        }
   
    然后修改 *subscribe* 方法以包括该区域设置：
   
        - (void) subscribeWithLocale: (int) locale categories:(NSSet *)categories completion:(void (^)(NSError *))completion{
            SBNotificationHub* hub = [[SBNotificationHub alloc] initWithConnectionString:@"<connection string>" notificationHubPath:@"<hub name>"];
   
            NSString* localeString;
            switch (locale) {
                case 0:
                    localeString = @"English";
                    break;
                case 1:
                    localeString = @"French";
                    break;
                case 2:
                    localeString = @"Mandarin";
                    break;
            }
   
            NSString* template = [NSString stringWithFormat:@"{\"aps\":{\"alert\":\"$(News_%@)\"},\"inAppMessage\":\"$(News_%@)\"}", localeString, localeString];
   
            [hub registerTemplateWithDeviceToken:self.deviceToken name:@"localizednewsTemplate" jsonBodyTemplate:template expiryTemplate:@"0" tags:categories completion:completion];
        }
   
    请注意，我们现在使用的是 *registerTemplateWithDeviceToken* 方法而非 *registerNativeWithDeviceToken*。 当我们注册一个模板时，必须提供 json 模板，还要指定其名称（因为我们的应用程序可能要注册不同的模板）。 确保将类别作为标记注册，因为我们要确保接收有关这些新闻的通知。
   
    添加一个方法以从用户默认设置中检索区域设置：
   
        - (int) retrieveLocale {
            NSUserDefaults* defaults = [NSUserDefaults standardUserDefaults];
   
            int locale = [defaults integerForKey:@"BreakingNewsLocale"];
   
            return locale < 0?0:locale;
        }
2. 现在我们修改了 Notifications 类，必须确保 ViewController 使用新的 UISegmentControl。 在 *viewDidLoad* 方法中添加以下行，以确保显示当前选择的区域设置：
   
        self.Locale.selectedSegmentIndex = [notifications retrieveLocale];
   
    然后，在 *subscribe* 方法中，将对 *storeCategoriesAndSubscribe* 的调用更改为：
   
        [notifications storeCategoriesAndSubscribeWithLocale: self.Locale.selectedSegmentIndex categories:[NSSet setWithArray:categories] completion: ^(NSError* error) {
            if (!error) {
                UIAlertView *alert = [[UIAlertView alloc] initWithTitle:@"Notification" message:
                                      @"Subscribed!" delegate:nil cancelButtonTitle:
                                      @"OK" otherButtonTitles:nil, nil];
                [alert show];
            } else {
                NSLog(@"Error subscribing: %@", error);
            }
        }];
3. 最后，必须在 AppDelegate.m 中更新 *didRegisterForRemoteNotificationsWithDeviceToken* 方法，以便在应用启动时正确刷新你的注册信息。 将对通知的 *subscribe* 方法的调用更改为：
   
        NSSet* categories = [self.notifications retrieveCategories];
        int locale = [self.notifications retrieveLocale];
        [self.notifications subscribeWithLocale: locale categories:categories completion:^(NSError* error) {
            if (error != nil) {
                NSLog(@"Error registering for notifications: %@", error);
            }
        }];

## <a name="optional-send-localized-template-notifications-from-net-console-app"></a>（可选）通过 .NET 控制台应用发送本地化的模板通知。
[!INCLUDE [notification-hubs-localized-back-end](../../includes/notification-hubs-localized-back-end.md)]

## <a name="optional-send-localized-template-notifications-from-the-device"></a>（可选）通过设备发送本地化的模板通知
如果你无权访问 Visual Studio，或者只是想要试一试直接从设备上的应用发送本地化的模板通知。  那么，你只需将本地化的模板参数添加到你在前一教程中定义的 `SendNotificationRESTAPI` 方法。

        - (void)SendNotificationRESTAPI:(NSString*)categoryTag
        {
            NSURLSession* session = [NSURLSession sessionWithConfiguration:[NSURLSessionConfiguration
                                     defaultSessionConfiguration] delegate:nil delegateQueue:nil];

            NSString *json;

            // Construct the messages REST endpoint
            NSURL* url = [NSURL URLWithString:[NSString stringWithFormat:@"%@%@/messages/%@", HubEndpoint,
                                               HUBNAME, API_VERSION]];

            // Generated the token to be used in the authorization header.
            NSString* authorizationToken = [self generateSasToken:[url absoluteString]];

            //Create the request to add the template notification message to the hub
            NSMutableURLRequest *request = [NSMutableURLRequest requestWithURL:url];
            [request setHTTPMethod:@"POST"];

            // Add the category as a tag
            [request setValue:categoryTag forHTTPHeaderField:@"ServiceBusNotification-Tags"];

            // Template notification
            json = [NSString stringWithFormat:@"{\"messageParam\":\"Breaking %@ News : %@\","
                    \"News_English\":\"Breaking %@ News in English : %@\","
                    \"News_French\":\"Breaking %@ News in French : %@\","
                    \"News_Mandarin\":\"Breaking %@ News in Mandarin : %@\","
                    categoryTag, self.notificationMessage.text,
                    categoryTag, self.notificationMessage.text,  // insert English localized news here
                    categoryTag, self.notificationMessage.text,  // insert French localized news here
                    categoryTag, self.notificationMessage.text]; // insert Mandarin localized news here

            // Signify template notification format
            [request setValue:@"template" forHTTPHeaderField:@"ServiceBusNotification-Format"];

            // JSON Content-Type
            [request setValue:@"application/json;charset=utf-8" forHTTPHeaderField:@"Content-Type"];

            //Authenticate the notification message POST request with the SaS token
            [request setValue:authorizationToken forHTTPHeaderField:@"Authorization"];

            //Add the notification message body
            [request setHTTPBody:[json dataUsingEncoding:NSUTF8StringEncoding]];

            // Send the REST request
            NSURLSessionDataTask* dataTask = [session dataTaskWithRequest:request
                       completionHandler:^(NSData *data, NSURLResponse *response, NSError *error)
               {
               NSHTTPURLResponse* httpResponse = (NSHTTPURLResponse*) response;
                   if (error || httpResponse.statusCode != 200)
                   {
                       NSLog(@"\nError status: %d\nError: %@", httpResponse.statusCode, error);
                   }
                   if (data != NULL)
                   {
                       //xmlParser = [[NSXMLParser alloc] initWithData:data];
                       //[xmlParser setDelegate:self];
                       //[xmlParser parse];
                   }
               }];

            [dataTask resume];
        }




## <a name="next-steps"></a>后续步骤
有关使用模板的详细信息，请参阅：

* [使用通知中心通知用户：ASP.NET]
* [使用通知中心通知用户：移动服务]

<!-- Images. -->

[13]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized1.png
[14]: ./media/notification-hubs-ios-send-localized-breaking-news/ios_localized2.png






<!-- URLs. -->
[如何：服务总线通知中心（iOS 应用）]: http://msdn.microsoft.com/library/jj927168.aspx
[使用通知中心发送突发新闻]: /manage/services/notification-hubs/breaking-news-ios
[移动服务]: /develop/mobile/tutorials/get-started
[使用通知中心通知用户：ASP.NET]: /manage/services/notification-hubs/notify-users-aspnet
[使用通知中心通知用户：移动服务]: /manage/services/notification-hubs/notify-users
[提交应用页]: http://go.microsoft.com/fwlink/p/?LinkID=266582
[我的应用程序]: http://go.microsoft.com/fwlink/p/?LinkId=262039
[Live SDK for Windows]: http://go.microsoft.com/fwlink/p/?LinkId=262253
[移动服务入门]: /develop/mobile/tutorials/get-started/#create-new-service
[数据处理入门]: /develop/mobile/tutorials/get-started-with-data-ios
[身份验证入门]: /develop/mobile/tutorials/get-started-with-users-ios
[推送通知入门]: /develop/mobile/tutorials/get-started-with-push-ios
[向应用用户推送通知]: /develop/mobile/tutorials/push-notifications-to-users-ios
[使用脚本为用户授权]: /develop/mobile/tutorials/authorize-users-in-scripts-ios
[JavaScript 和 HTML]: ../get-started-with-push-js.md

[面向移动服务的 Windows 开发者预览版注册步骤]: ../mobile-services-windows-developer-preview-registration.md
[wns 对象]: http://go.microsoft.com/fwlink/p/?LinkId=260591
[通知中心指南]: http://msdn.microsoft.com/library/jj927170.aspx
[适用于 iOS 的通知中心操作方法指南]: http://msdn.microsoft.com/library/jj927168.aspx



<!--HONumber=Nov16_HO3-->


