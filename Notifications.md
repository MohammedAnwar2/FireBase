
# الدرس الأول 

 هناك طريقتين لإرسال الاشعار.
 
1- token. 

2- topic.

-أولا بإستعمال ال Token

-هي التي تتظمن اننا أحصل على ال token الخاص بالجهاز وهذا ال token ما يتشابه مع اي token في جهاز آخر ، يعني لكل جهاز token خاص به ، لذا راح يتم إرسال الأشعار بالاعتماد على ال token.

* كيفية الحصول على ال token


للحصول على ال token نستخدم الكود البرمجي التالي على حسب الموقع الرسمي flutterFire

```dart
  void getToken()async{
    var token =await FirebaseMessaging.instance.getToken();
  }
  @override
  void initState() {
    super.initState();
    getToken();
  }
```
متى يظهر الأشعار 

الأشعار راح يوصل بكل الحالات( مفتوح" Foreground " مغلق " Terminated" او background ) لكن بيظهر  الإشعار في حالة إذا كان التطبيق مغلق(Terminated) او في حالة ال background ولن يظهر في حالة كان التطبيق مفتوح لكن توجد فنكشن معينة تساعد على استعمال هذا الأشعار داخل التطبيق.

راح يتم استعمال الأشعار المرسل في حالة كان التطبيق مفتوح باستعمال ال local notification.
راح نستخدم ال local notification مع ال firebase massaging لإظهار الأشعار في حالة كان التطبيق مفتوح.

 
-متى يتغير ال token ?

يتغير في حالة حذفنا التطبيق وثبتناه من جديد.
لذا يجب اخذ ال token في كل عملية تسجيل دخول اي راح نعمل refresh لل token القديم في قاعدة البيانات المستخدمة( سواءا في الفايربيس او قواعد البيانات او php او Njs ) 

--------------------------------------------------

# الدرس الثاني


- كيف ممكن نعطي للتطبيق الاذن الوصول الى الاشعارات
- الدرس السابق الاشعار ظهر عندنا بدون اي مشاكل وبدون حتى ما اضطر اضيف ال permissions وذلك لاننا نستخدم نظام الاندرويد . وباغلب الاوقات يكون ال permissions ممنوح اي متاح ، بينما في حالة ال apple وال web نحتاج الى اضافة ال permissions حتى يقدر التطبيق للوصول الى اذن الاشعارات.
- الافضل اضيف ال permissions في الاندرويد برضو من شان ما اواجه اي مشاكل .

راح نضيف ال function التالية من اجل الوصول الى permissions ، لكن هذه ال function مخصصه الى ال web and ios.اما الاندرويد فهو دايركت متاح له ال permissions
```dart
  void myPermission()async{
    FirebaseMessaging messaging = FirebaseMessaging.instance;

    NotificationSettings settings = await messaging.requestPermission(
      alert: true,
      announcement: false,
      badge: true,
      carPlay: false,
      criticalAlert: false,
      provisional: false,
      sound: true,
    );

    if (settings.authorizationStatus == AuthorizationStatus.authorized) {
      print('User granted permission');
    } else if (settings.authorizationStatus == AuthorizationStatus.provisional) {
      print('User granted provisional permission');
    } else {
      print('User declined or has not accepted permission');
    }
  }

  @override
  void initState() {
    super.initState();
    myPermission();
  }
```
--------------------------------------------------

# الدرس الثالث

-كيف يمكننا ارسال الاشعارات من التطبيق بدلا من واجهة الفايربيس ? 

باستخدام ال Api  نقدر نرسل اشعارات الى المستخدمين وهذه هي الطريقة المتبعة
ندخل الى محرك البحث ونبحث عن how to send notification using api , روح مباشرة الى ال stackOverFlow
الرابط https://stackoverflow.com/questions/37490629/firebase-send-notification-with-rest-api 
 
بعدها استخدم ال postman لتجربة ال api ومن ثم حول ال api الى code مكتوب بلغة ال dart ,
الفنكش بتكون  كالتالي
```dart
RequestNotifications({required String title,required String body,required String token})async{
  String serverToken = 'AAAAxfKVswM:APA91bFoUNmeVb2PXuh1rmSR6KZK0uN9K3dRqNGT2GlCjBK-SRzVHNusfHgOO0lF0z97fme2zjzWXlamdhblPeRPExQscSNxwdokr9eETTXmxt4_Q-XRJ_WYszoOrmyak3ZxRBP0qtfg';
  var headers = {
    'Content-Type': 'application/json',
    'Authorization':'key=$serverToken'
  };
  var request = http.Request('POST', Uri.parse('https://fcm.googleapis.com/fcm/send'));
  request.body = json.encode({
    "to": token,
    "notification": {
      "title": title,
      "body": body,
      "mutable_content": true,
      "sound": "Tri-tone"
    }
  });
  request.headers.addAll(headers);
  http.StreamedResponse response = await request.send();
  if (response.statusCode == 200) {
    print(await response.stream.bytesToString());
  }
  else {
  print(response.reasonPhrase);
  }
}
```
ملاحظات مهمه جدا :
1 - ال body , title هما عنوان ومضمون الأشعار
2- ال token  راح يختلف من جهاز الى اخر لذا هو مطلوب بشكل دائم
3-ال serverToken هذا نجيبة من ال cloud Messagibg بعد ما نعمل لها enable 

--------------------------------------------------

# الدرس الرابع
- Foreground messages OnMessage
ال OnMessageهي عبارة عن فنكشن بتشتغل في حالة ال foreground( التطبيق مفتوح ) عند وصول الأشعار.
الفنكشن من نوع streem يعني بتلاحظ التعيرات مباشرة ، لذا افضل مكان لها بيكون في initState حق دالة ال main ، والافضل في الصفحة الرئيسية اللي بعد دالة ال main لاننا خارج ال materiaApp لذا الافضل نكون داخل ال MateriaApp من اجل تجنب حدوث اي خطأ.
```dart
void initState() {

super.initState();
 FirebaseMessaging.onMessage.listen((RemoteMessage message) {
if (message.notification != null) {
             Get.snackbar(message.notification!.title.toString(), message.notification!.body.toString());

      }
    });
  }
```
بداخل الفنكشن فينا نظهر Alert او dialog او sanckBare او local notification او نضيف فنكشن تعدل او تضيف على الداتا بيس او فنكشن تعمل refresh للتطبيق كل هذه الامور نقدر نظيفها داخل OnMessage اللي بيتم استدعاءها عند وصول اي اشعار.
 
--------------------------------------------------

# الدرس الخامس

Terminated Background  OnBackgroubdMessage
ال OnBackgroubdMessage هي عبارة عن فنكشن بتشتغل في حالة ال Terminated or Background( التطبيق مغلق او في الخلفية ) عند وصول الأشعار.

الفنكشن من نوع streem يعني بتلاحظ التعيرات مباشرة.
يوجد شرطين لهذه الفنكشن
1- It must not be an anonymous function.
2- It must be a top-level function (e.g. not a class method which requires initialization)

الشرط الثاني ، لازم تكون فوق كل ال functions حق التطبيق لذا بنحطها فوق دالة ال main
بالشكل التالي
  ```dart
Future<void> _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
  print("--------------onBackgroundMessage--------------------");
  print(message.notification!.title.toString());
  print(message.notification!.body.toString());
  print("---------------onBackgroundMessage-------------------");
}

void main() async {
   await Firebase.initializeApp(
    options: DefaultFirebaseOptions.currentPlatform,
  );
  FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);

  runApp(NoteAPP());
}
```
--------------------------------------------------

# الدرس السادس

ارسال data اضافية غير ال title and body مع ال notification.
 
و ذلك باضافة ال data في ال api وبداخلها Map وبداخلها نقدر نحط عدد لا نهائي من ال data.
بالشكل ذا
```dart
{
    "to": "cGebL2CcSpeKBglsRPQO3v:APA91bEtDvCDoA_8wRIH_8SJCkZb1Bk9E5Z3VKSicKcGSNV5kruNd9McQ2iRQ_Hc7nG8MLmnA5GIvOAegVTcq2aGKCW7vI1Tz4Uvr0ZU6FyM0cE_Qwic2TdVqa9t5CCOgHJrRJjv0zNz",
    "notification": {
        "title": "HI",
        "body": "MOHAMMED ANWAE",
        "mutable_content": true,
        "sound": "Tri-tone"
    },

    "data": {
    "name": "MOHAMMED ANWAE",
    "age": "23",
    "phone":"Ridmi Note 10"
      }

}
```
والفنكشن حقها بتكون كذا
```dart
RequestNotifications({required String title,required String body,required String token})async{
  String serverToken = 'AAAAxfKVswM:APA91bFoUNmeVb2PXuh1rmSR6KZK0uN9K3dRqNGT2GlCjBK-SRzVHNusfHgOO0lF0z97fme2zjzWXlamdhblPeRPExQscSNxwdokr9eETTXmxt4_Q-XRJ_WYszoOrmyak3ZxRBP0qtfg';
  var headers = {
    'Content-Type': 'application/json',
    'Authorization':'key=$serverToken'
  };
  var request = http.Request('POST', Uri.parse('https://fcm.googleapis.com/fcm/send'));
  request.body = json.encode({
    "to": token,
    "notification": {
      "title": title,
      "body": body,
      "mutable_content": true,
      "sound": "Tri-tone"
    },
    
    "data": {
      "name": "MOHAMMED ANWAE",
      "age": "23",
      "phone": "Ridmi Note 10"
    }
    
  });
  request.headers.addAll(headers);
  http.StreamedResponse response = await request.send();
  if (response.statusCode == 200) {
    print(await response.stream.bytesToString());
  }
  else {
  print(response.reasonPhrase);
  }
}
```
- الان بدل ماهم المطاليب الاساسية title, body and token , الان فيك تظيف data على حسب ما تريد.


--------------------------------------------------

# الدرس السابع

Handling Interaction
كيفية التفاعل مع الاشعارات.

عند وصول الاشعار والضغط علية بيفتح التطبيق مباشرة بدون عمل اي حاجه وهذا هو الوضع الافتراضي.
لكن لو نحن عايزين عند الضغط( و ليس عند وصول الاشعار ) على الاشعار الواصل الينا ان يحدث شيء ما مثل استدعاء فنكشن محدده او شيء من هذا القبيل.

يوجد  functions  2 تتفاعل مع الاشعارات

The firebase-messaging package provides two ways to handle this interaction:

1- getInitialMessage(): 
If the application is opened from a terminated state a Future containing a RemoteMessage will be returned. Once consumed, the RemoteMessage will be removed.
عندما يكون التطبيق مغلق وضغطنا على الاشعار ، راح تتفعل هذه الفنكشن.

2- onMessageOpenedApp: 
A Stream which posts a RemoteMessage when the application is opened from a background state.
عندما يكون التطبيق في الخلفية اي ال background وضغطنا على الاشعار ، راح تتفعل هذه الفنكشن


1- الان راح نتعرف على ال onMessageOpenedApp
مثل ما قلنا بتشتغل عند الضغط على الإشعار في حالة كان التطبيق في وضع ال background فان هذه الفنكشن تشتغل , ومن ابرز استخداماتها الإنتقال الى صفحه معينه عند الضغط على الإشعار الواصل إلينا.
- هذه عبارة عن stream
مثال من ال documentation 🥰
```dart
  void initState() {

    super.initState();
    FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage event) {

      if(event.data["type"]=="chat")
      {
        Get.to(()=>ChatPage());
      }
    });

}
```
في ذا المثال اولا استفدنا من ال data الاضافية اللي بعثناها مع الإشعار من اجل عملية التوجة الى صفحة اخرى.

ملاحظه مهمه ، حاول تكون هذه الفنكشن في اول صفحه بالتطبيق ولا تخليها في دالة ال main تجنبا للمشاكل 


2- الان راح نتعرف على ال getInitialMessage()
مثل ما قلنا بتشتغل عند الضغط على الإشعار في حالة كان التطبيق في وضع ال Terminated (اي التطبيق مغلق) فان هذه الفنكشن تشتغل , ومن ابرز استخداماتها الإنتقال الى صفحه معينه عند الضغط على الإشعار الواصل إلينا.
- هذه عباره عن فنكشن. نسويها برع بعدين نحطها في دالة ال  initState
مثال من ال documentation 🥰
```dart
  Future<void> setupInteractedMessage() async {
    RemoteMessage? initialMessage = await FirebaseMessaging.instance.getInitialMessage();
    if(initialMessage!=null)
    {
      if(initialMessage.data["type"]=="chat")
      {
        Get.to(()=>ChatPage());
      }
    }
  }


  void initState() {
    super.initState();
    setupInteractedMessage();

}
```
- تجربتي ---» getInitialMessage() ، لما افتح التطبيق اول مره واغلقه اي اخليه بوضع ال Terminated واحاول استلم اشعار فاننا لا اقدر ، ولحل هذه المشكله لما نشغل التطبيق اول راح نظهر snackbare مخفي لانه اذا سوينا كذا خلاص كأن ال getInitialMessage() يتم تفعيلها

--------------------------------------------------

# الدرس الثامن

- خلال الدرس الأول شرحنا أنه يمكن بواسطة الtoken إرسال اشعار ، وبرضو عرفنا ان ال token يتغير عند حذف التطبيق و تنزيله وبرضو لكل جهاز token فريد اي مختلف ، إذن فال token هو فريد unique , فبالتالي إذا أنا عايز ارسل اشعار الى شخص معين فانه لازم أعرف ال token حق الجهاز بتاعه حتى ارسل الأشعار ، افترض ان هذا الشخص له اكثر من جهاز وكل الأجهزة تبعه فيها نفس الحساب فالبتالي يجي علي معرفة كل ال token حق جميع الأجهزة حتى يتم إرسال الأشعار لكل الأجهزة بتاعه لان الشخص يقدر يفتح له حساب من اكثر من جهاز وذي مشكله كبيرة انه يجب علينا معرفة ال token حق الأجهزة حق المستخدم. مثال بيوضح كل حاجة...» افترض انه معنا 5000 مستخدم وعايز ارسل لهم نفس الأشعار ، فهل أنا باروح الى الفايرستور وبعمل استعلام لكل ال tokens المخزنين عندي في الفايرستور وارسل اشعار واحد واحد ؟؟ و ذا الشيء مش منطقي بالخالص ، لانه بضطر أعمل for وبعمل اكثر من request وكل ذا محسوب عليّ كمبرمج وبرضو برهق ال server كثيير .

الحل لكل هذا , ال Topic
مثال لتوضيح ال topic
- إذا نزل احد اليوتيوبر اللي نحن مشتركين معه post معين فسيصل هذا الأشعار للجميع ، هنا تم استخدام ال topic
- افترض ان ال topic حق اليوتيوبر هذا اسمه Mohammad Anwar , 

```dart
await FirebaseMessaging.instance.subscribeToTopic('Mohammad Anwar');
```
فهنا أي شخص مشترك بهذه القناة راح يوصل له اشعار ، ما بيهمنا ال token بتاعه.
- فال topic هو نفس نظام قنوات اليوتيوب او الفيسبوك، إذا كان مشترك راح يوصل له اشعار واذا كان غير مشترك ما رح يوصل له اشعار ، يعني لو مثلا أنا مشترك في قناة فيسبوك ،وصاحب القناة ارسل post فانا الأشعار راح يوصل لكل المستخدمين المشتركين بهذه القناة.



- برنامج كامل لإرسال الاشعارات مع الشرح👇🏻👇🏻
  
```dart
//Function to send notificatoins

RequestNotificationsTopic({required String title,required String body,required String topic})async{
  String serverToken = 'AAAAxfKVswM:APA91bFoUNmeVb2PXuh1rmSR6KZK0uN9K3dRqNGT2GlCjBK-SRzVHNusfHgOO0lF0z97fme2zjzWXlamdhblPeRPExQscSNxwdokr9eETTXmxt4_Q-XRJ_WYszoOrmyak3ZxRBP0qtfg';
  var headers = {
    'Content-Type': 'application/json',
    'Authorization':'key=$serverToken'
  };
  var request = http.Request('POST', Uri.parse('https://fcm.googleapis.com/fcm/send'));
  request.body = json.encode({
    "to": "/topics/$topic",
    "notification": {
      "title": title,
      "body": body,
      "mutable_content": true,
      "sound": "Tri-tone"
    },

    "data": {
      "name": "MOHAMMED ANWAE",
      "age": "23",
      "phone": "Ridmi Note 10"
    }

  });
  request.headers.addAll(headers);
  http.StreamedResponse response = await request.send();
  if (response.statusCode == 200) {
    print(await response.stream.bytesToString());
  }
  else {
  print(response.reasonPhrase);
  }
}

```

```dart
class TestPage extends StatelessWidget {
  const TestPage({super.key});
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Notifications'),
      ),
      body: ListView(
        children: [
          ElevatedButton(onPressed: () async{
            // subscribe to topic on each app start-up
            await FirebaseMessaging.instance.subscribeToTopic('b');
          }, child: const Text("SubscribeToTopic")),
          ElevatedButton(onPressed: () async{
            await FirebaseMessaging.instance.unsubscribeFromTopic('b');

          }, child: const Text("UnsubscribeToTopic")),
          ElevatedButton(onPressed: () async{

            await RequestNotificationsTopic(title: "See the Error",body: "Mohammed Anwar",topic: 'b');
          }, child: const Text("Send Notification")),

        ],
      ),
    );
  }
}

```

في البرنامج اعلاه.
-أولا الفنكشن  RequestNotificationsTopic
تحتوي على ثلاثة براميتر title , body , topic
هي هي نفسها الفانكشن اللي نرسل بواسطها الاشعار بواسطة ال token , لكن هوا بدل ما نرسل بواسطة token ,نرسل بواسطة ال topic . يعني بدل ما to الى token الجهاز ، فنعمل to الى /topics/اسم الtopic .

ملاحظة مهمه لازم يكون كذا 
"to": "/topics/$nameOfTopic",


ثانيا الازرار في البرنامج.

الزر الأول : 
FirebaseMessaging.instance.subscribeToTopic('b')
وظيفتة يخلي المستخدم يشترك في ال topic اللي اسمه هنا 'b' ، يعني كلما ارسلنا اشعار على ال topic اللي اسمه b راح يستلمه المستخدم اللي اشترك على هذا ال topic.

الزر الثاني :
FirebaseMessaging.instance.unsubscribeFromTopic('b')
وظيفتة إلغاء الاشتراك من  ال topic اللي اسمه 'b' ، يعني كلما ارسلنا اشعار على ال topic اللي اسمه b ما راح يستلمه المستخدم ابدا ، لانه عاده ما  اشترك على هذا ال topic.

الزر الثالث : 

await RequestNotificationsTopic(title: "See the Error",body: "Mohammed Anwar",topic: 'b');
وظيفته استدعاء الفنكشن RequestNotificationsTopic من أجل إرسال الأشعار.



ملاحظة مهمه جدا.

كل الدوام اللي تكملنا عليها في ال token , برضو نقدر نطبقها على ال topic الاختلاف فقط هو طريقة إرسال الأشعار ، يعني اما بال token او ال topic , وكل الفنكشنات تشتغل على كلا الطريقتين ....فالنكشنات مثل ما تكملنا عليهن هن👇🏻👇🏻

OnMessage
OnBackgroubdMessage
onMessageOpenedApp
getInitialMessage()
