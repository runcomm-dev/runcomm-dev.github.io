# TouchAd 소개
* 매체사 제휴서비스인 런컴광고전송플랫폼 서비스(이하, 터치애드) 안드로이드 앱용 SDK 설치에 관한 내용입니다.
* 터치애드 서비스는 광고 포인트 적립 플랫폼입니다.
* 제휴앱 회원이 터치애드를 통해 광고를 이용할 경우 포인트 적립이 됩니다.
* 사용자가 광고 이용 후 포인트 적립이 이루어집니다.

  ## 광고전송 SDK 주요기능
  * 모바일 웹 광고 화면 : 일반적인 광고적립 앱 서비스(예:캐시슬라이드)에 사용되는 광고목록화면
  * FCM Push 기능 : 적립, 통신비 차감, 전면광고 결과화면

   ## 매체사 앱 SDK 연동을 위한 업무진행 절차
  * 광고 SDK에서 발급하는 platform id값을 획득하여 앱프로젝트 코딩에 사용.
  * 광고 SDK 라이브러리(.aar) 앱프로젝트 Import.
  * 추가코딩 (SDK 초기화, 전면광고적용, FCM 리시버)
  * FCM 전송 Public Web API(POST 방식) 제작후 광고 SDK 담당자에게 전달
  * 테스트용 apk 파일 광고 SDK 담당자에게 전달
  * 광고 SDK 기능테스트 (가입, 적립, 차감, FCM)
  * 상세한 기술적 내용은 아래 TouchAd SDK 구성, TouchAd SDK 설치 가이드 항목을 참고하시기 바랍니다.


# TouchAd SDK For KB 구성

* KB 버전 터치애드 SDK에 대한 설명입니다.
* 터치애드 SDK For KB 리브메이트 앱은 안드로이드 스튜디오(4.0.1)으로 개발되었습니다.
* SDK 결과물은 확장자 aar 형태로 별도 제공됩니다.
* 안드로이드 minSdkVersion : 17 , targetSdkVersion : 30, compileSdkVersion : 30 (으)로 빌드되었습니다.



### SDK build.gradle(app)

* 터치애드 SDK는 <b>http라이브러리(retrofit2, OkHttp3), 자바비동기 이벤트 기반 라이브러리(rxjava2)</b>를 사용합니다.
* buildTypes안에 proguard에 대한 debug와 release에 따른 동작, stacktrace에 대한 예외처리를 위한 buildConfigField를 설정하였습니다.
* buildTypes에 consumerProguard File을 사용하여 라이브러리 프로젝트에서 난독화 규칙을 제공하여 매체사 앱 프로젝트에 자동으로 규칙이 적용 됩니다.
* 아래는 SDK 프로젝트 build.gradle(app)에 실제 적용된 내용입니다.

~~~
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'

android {

    compileSdkVersion 30

    defaultConfig {
        minSdkVersion 17
        targetSdkVersion 30
        versionCode 1009
        versionName "1.0"
        multiDexEnabled true

    }

    buildTypes {
        debug {
            minifyEnabled false
            useProguard false
            buildConfigField "boolean", "TraceEnable", "true"
            buildConfigField "boolean", "TraceVerbose", "true"
            buildConfigField "java.util.Date", "buildTime", "new java.util.Date(" +
                    System.currentTimeMillis() + "L)"
            consumerProguardFile 'proguard-rules.pro'

        }
        release {
            minifyEnabled false
            buildConfigField "boolean", "TraceEnable", "false"
            buildConfigField "boolean", "TraceVerbose", "false"
            buildConfigField "java.util.Date", "buildTime", "new java.util.Date(" +
                    System.currentTimeMillis() + "L)"
            consumerProguardFile 'proguard-rules.pro'
        }
    }
    //라이브러리 모듈에 Flavor를 추가했을 경우 아래 옵션으로 기본 명시를 해 주어야
    // Android Studio에서 Run이 정상 동작을 한다.
    //Error:All flavors must now belong to a named flavor dimension. The flavor 'flavor_name' is not assigned to a flavor dimension.
    /*flavorDimensions "flavors"
    productFlavors{
        dev{
            dimension "flavors"
        }
        product{
            dimension "flavors"
        }

    }*/

    compileOptions {
        sourceCompatibility 1.8
        targetCompatibility 1.8
    }

    lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
}

dependencies {
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.72"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation 'com.squareup.retrofit2:retrofit:2.5.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.5.0'
    implementation 'com.squareup.okhttp3:okhttp:3.12.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.12.0'
    implementation 'com.google.firebase:firebase-core:17.4.3'
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
    implementation 'com.auth0.android:jwtdecode:2.0.0'
}
~~~



### AndroidManifest.xml

* SDK 내부에 사용되는 resource 아이디는 APK와 충돌하지 않게 네이밍 합니다.
* 아래에 권한설정 내용에 주석으로 권한 내용과 권한레벨을 작성하였으니 참고하시면 됩니다.
* 권한 내용 중 SYSTEM_ALERT_WINDOW와 같이 **특별권한 런타임 레벨**은 터치애드 메인 화면에 진입하는 액티비티에서 checkRequiredPermission() 함수를 통해 다른 앱 위에 그리기 권한을 요청합니다. 
    **사용자**가 수락할 경우 앱의 모든 기능이 정상적으로 동작하며, 권한을 거부할 경우 해당권한이 필요한 기능이 동작하지 않습니다.
* 아래는 소스코드 레벨에서 권한을 설정한 내용으로 특별 권한 레벨 설정 예시입니다.
~~~
private fun checkRequiredPermission() {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
        val canDrawble = Settings.canDrawOverlays(context)
        if (!canDrawble) {
            val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:$packageName"))
            startActivityForResult(intent, REQ_CODE_OVERLAY_PERMISSION)
        }
    }
}
~~~




* 참조용으로 SDK 내에 설정된 내용입니다.
~~~
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="kr.co.touchad.sdk">

    <!--인터넷 접속(네트워크 작업)을 위한 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="android.permission.INTERNET"/>

    <!--네트워크 연결확인을 위한 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />

    <!--디바이스 진동사용 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="android.permission.VIBRATE" />

    <!--어플리케이션이 항상 켜져있도록 하는 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="android.permission.WAKE_LOCK" />

    <!--죽지 않는 서비스를 구현하기 위한 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
	
	<!--Task 정보를 구하는 권한, 21버전 미만 통화상태 확인하기 위해 사용 // 권한레벨 : signatureOrSystem-->
    <uses-permission android:name="android.permission.GET_TASKS"/>

    <!--Android 10(API 29) 이상에서 전체화면 활동 실행 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />

    <!--다른 앱 위에 그리기 권한 // 권한 레벨 : 특별-->
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />

    <!--진동 사용 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name = "android.permission.VIBRATE"/>

    <!--전화관련 정보 읽기 권한 // 권한 레벨 : 위험-->
    <uses-permission android:name="android.permission.READ_PHONE_STATE"/>

    <application
        android:icon="@mipmap/tc_ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/tc_ic_launcher_round"
        android:supportsRtl="true"
        android:allowBackup="false"
        android:usesCleartextTraffic="true"
        android:hardwareAccelerated="true"
        android:theme="@style/TouchAdTheme">

        <!--CPI 광고 처리를 위한 서비스 -->
        <service
            android:name="kr.co.touchad.sdk.ui.service.TouchAdService"
            android:enabled="true">
            <intent-filter>
                <action android:name="kr.co.touchad.ui.service" />
                <category android:name="android.intent.category.DEFAULT" />
            </intent-filter>
        </service>

        <!-- 웹뷰화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.webview.WebViewActivity"
            android:theme="@style/TouchAdTheme">
        </activity>

        <!-- 전면 광고 화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.advertise.AdFullActivity"
            android:theme="@style/TouchAdTheme">
        </activity>

        <activity android:name="io.card.payment.DataEntryActivity"/>
        
    </application>
</manifest>
~~~



### proguard-rules.pro 파일

* jar형태로 구성된 라이브러리와 달리, **aar로 배포되는 터치애드 SDK는 난독화 규칙을 포함하여 배포할 수 있습니다.** proguard-rules.pro 파일에 아래 내용이 추가되었으며 매체사 앱 측에서 별도로 **SDK에 대한 난독화 규칙을 추가하지 않습니다.**
* 추가로 retrofit2및 glide, stactrace 오류보고에 대한 난독화 예외도 추가합니다.
* 아래 코드는 SDK에 추가된 Proguard-rules.pro에 대한 내용입니다.
~~~
-keep class kr.co.touchad.** {public *;}#패키지 하위 클래스 중 public 메소드만 난독화x
-keep class android.support.** { *; }
-keep class com.google.** { *; }
-keepparameternames#파라미터 이름을 난독화x

#소스 파일의 라인을 섞지 않는 옵션 ( 안하게되면 나중에 stacktrace보고 어느 line에서 오류가 난 것인지 확인 불가)
-keepattributes SourceFile,LineNumberTable

# retrofit2
-dontwarn okhttp3.**
-dontwarn okio.**
-dontwarn retrofit2.**
-dontnote okhttp3.**
-dontnote retrofit2.Platform
-dontnote retrofit2.Platform$IOS$MainThreadExecutor
-dontwarn retrofit2.Platform$Java8
-keepattributes Signature
-keepattributes Exceptions

# glide
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep public class * extends com.bumptech.glide.module.AppGlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
 **[] $VALUES;
 public *;
}
-keepclassmembers enum * {
   public static **[] values();
   public static ** valueOf(java.lang.String);
}
~~~



### styles.xml 파일

* 터치애드는 메인 테마로 AppCompat.Light.NoActionBar를 사용하며 테마 네이밍이 겹치지 않도록 주의합니다.

* 터치애드 메인 테마명 : name = TouchAdTheme



### Foreground Service

* 터치애드는 CPI 광고(Cost Per Install) 설치 체크를 위해 CPI 광고 참여시 background service를 시작합니다.

* targetSdkVersion 26부터 적용되는 background service 실행 제한정책으로 해당 service가 background -> foreground로 변경되었습니다.

* Foreground Service를 시작하기 위해서 앱은 해당 서비스가 백그라운드로 실행되고 있다는것을 사용자에게 알려야하며, 해당 알림은 알림창의 notification 형태로 노출됩니다.

* 터치애드 foreground service 시작시 아래와 같은 notification이 알림창에 노출됩니다.

     ![그리기이(가) 표시된 사진  자동 생성된 설명](https://lh5.googleusercontent.com/GHExixOTfH_DnR__bCkyYx8roKyzjRN1jSPWfysLy00C44CT7P8OuGUyyOZgYyy8_ZF6Gc6Z-p0GmDs3aBCOzHiw_XKK_ZdRmVe7VEMFXRLG3X9Lebb4vRJ9rcDa6k3ztx5k7yBF)


#  TouchAd SDK For KB 설치 가이드

* 정상적인 제휴서비스를 위한 터치애드 SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 **touchad-sdk-1.0.0.aar** 파일을 프로젝트의 libs 폴더에 넣어줍니다.



## build.gradle 설정 

  1. **build.gradle(project)파일수정**
     *      * 광고Id를 가져와 터치애드 광고참여를 하기 위해 아래 dependencies의 calsspath에 google-services를 추가합니다.
            * allprojects안의 repositories에 maven내용을 추가합니다.
            * google-services 사용에 필요한 파일인 google-services.json파일은 샘플 프로젝트 내 gradle 파일과 같은 레벨에서 찾을 수 있습니다.
            * 아래는 실제 작성된 예시입니다.
~~~
// Top-level build file where you can add configuration options common to all sub-projects/modules.
buildscript {
   repositories {
       google()
       jcenter()
   }
   dependencies {
       classpath 'com.android.tools.build:gradle:4.0.1'
       classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:1.3.72"
       classpath 'com.google.gms:google-services:4.2.0'
       // NOTE: Do not place your application dependencies here; they belong
       // in the individual module build.gradle files
   }
}

allprojects {
   repositories {
       maven { url 'https://maven.google.com'}

       jcenter()
       google()
   }
}

task clean(type: Delete) {
   delete rootProject.buildDir
}
~~~

  2. **build.gradle(app)파일수정**
     *  아래 dependencies 영역내용을 추가합니다.
     *  build.gradle에  android{…}영역과 dependencies{…}사이에 repositories{flatDir{…}}을 추가합니다.
     *  dependencies 영역에 Implementation name: ’touchad-sdk-1.0.0’, ext: ’arr’를 추가합니다.
     *  중복된 내용은 생략 합니다.
~~~
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'com.google.gms.google-services'

android {
    compileSdkVersion 30

    defaultConfig {
        applicationId "kr.co.touchad"
        minSdkVersion 17
        targetSdkVersion 30
        versionCode 1009
        versionName "1.0"
        multiDexEnabled true
    }

    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility 1.8
        targetCompatibility 1.8
    }

    lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
}

repositories {
    flatDir{
        dirs 'libs'
    }
}

dependencies {
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.72"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'com.squareup.retrofit2:retrofit:2.5.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.5.0'
    implementation 'com.squareup.okhttp3:okhttp:3.12.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.12.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'

    implementation 'com.google.firebase:firebase-messaging:20.2.1'
    implementation 'com.google.firebase:firebase-core:17.4.3'
    implementation "androidx.viewpager2:viewpager2:1.0.0"
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
    implementation 'com.github.bumptech.glide:glide:4.8.0'

    implementation name: 'touchad-sdk-1.0.0', ext: 'aar'

    implementation 'com.makeramen:roundedimageview:2.3.0'
    implementation 'com.auth0.android:jwtdecode:2.0.0'
    implementation 'androidx.multidex:multidex:2.0.0'
}
~~~



## 터치애드 플랫폼 클래스 함수

   - 기능을 모듈화하여 Static 함수형태로 호출합니다.
   -  아래 간략한 설명입니다.

~~~
object TouchAdPlatform {  

/**
* 터치애드 전면광고 화면 시작
*/
fun  openKBAdvertise(context: Context, mbrId: String, data: String)

/**
* 터치애드 화면 시작
*/
fun  openKBTouchAdMenu(context: Context, mbrId: String, adPushYn: String?, gender: String?, birthYear: String?, callback: (() -> Unit)?)

/**
* 참여적립 화면 시작
*/
fun  openKBEarningMenu(context: Context, mbrId: String, adPushYn: String?, gender: String?, birthYear: String?)
 
}
~~~



##  터치애드 전면광고 화면 시작

*  카드결제완료 후 푸시 수신하고 이때 터치애드의 전면광고 화면을 띄울 경우 호출합니다.
*  광고회원가입된 유저일경우 전면광고 화면으로 이동합니다.
*  mbrId = 고객관리번호(필수값)
*  data = 푸시데이터(필수값, FCM 항목 참고하시면 됩니다.)

*  아래는 터치애드 전면광고 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openKBAdvertise(context, mbrId, data);
~~~



##  참여적립 화면 시작

*  KB 앱 내에서 참여적립 메뉴를 선택하면 약관동의를 거치고 참여적립 화면을 시작할때 호출합니다.
*  mbrId = 고객관리번호(필수값)
*  아래 3개의 파라미터는 협의중인 단계로 Null 값을 넣어 호출해도 되는 내용입니다.
   *  adPushYn = KB 광고푸시수신여부 (대문자 "Y" 또는 "N")
   *  gender = 성별 1(남), 2(여), 3(2000년이후 남), 4(2000년이후 여)
   *  birthYear = 생년월일 YYMMDD 6자리
*  아래는 참여적립 화면 시작함수 호출 예시입니다.

~~~
/*
* 세번째 파라미터 : adPushYn
* 네번째 파라미터 : gender
* 다섯번째 파라미터 : birthYear
*/
TouchAdPlatform.openKBEarningMenu(context, mbrId, null, null, null)
~~~



## 터치애드 화면 시작

*  KB 앱 내에서 터치애드 메뉴를 선택하면 약관동의를 거치고 터치애드 화면을 시작할때 호출합니다.
*  mbrId = 고객관리번호(필수값)
*  아래 4개의 파라미터는 협의중인 단계로 Null 값을 넣어 호출해도 되는 내용입니다.
   *  adPushYn = KB 광고푸시수신여부 (대문자 "Y" 또는 "N")
   *  gender = 성별 1(남), 2(여), 3(2000년이후 남), 4(2000년이후 여)
   *  birthYear = 생년월일 YYMMDD 6자리
   *  kbCallback = KB 설정화면 호출을 위한 콜백변수 
   KB 앱 광고푸시 설정 화면을 오픈할수 있는 기능을 콜백 영역내 구현합니다.  (옵션)
*  아래는 터치애드 화면 시작함수 호출 예시입니다.

~~~
/*
* 세번째 파라미터 : adPushYn
* 네번째 파라미터 : gender
* 다섯번째 파라미터 : birthYear
* 여섯번째 파라미터 : kbCallback = KB 설정화면 호출을 위한 콜백 변수
*/
TouchAdPlatform.openKBTouchAdMenu(context, mbrId, null, null, null, null)
~~~

##  터치애드 푸시 수신 시

* 터치애드는 푸시 송신 시 KB 에서 제공한 Public API를 이용하여 Push(FCM)를 전송합니다.
* 매체사 앱 Push Receiver에서 터치애드 관련 Push 수신 시, Data Property에 존재하는 터치애드 관련 데이터를 Parsing하여 SDK에 전달합니다.
* 푸시관련에 대한 세부 내용은 아래 FCM 관련 부분을 참고바랍니다.



##  FCM 전송

* 터치애드는 푸시 송신 시 앱 제작사에서 제공한 Public API를 이용하여 데이터를 전달하고 제작사 서버에서 앱으로 Push(FCM)을 전송하는 형태입니다.(협의중)
* Public API를 개발하신 후 광고 SDK 담당자에게 전달바랍니다.
* 요청 데이터 형식(key : touchad, value : 문자열)
~~~
%7B%22touchad%22%3A%22touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fkb%3FonOff%3D1%26cd%3D1916%26cardIdx%3D896%22%7D
~~~

* API를 통해 POST된 데이터를 FCM 데이터의 구성요소 중 data 프로퍼티에 담아서 FCM 전송 바랍니다. (* 변경 가능성 있습니다.)

* FCM 전송 포맷 예시
~~~
{
  "android": {
    "priority": "high",
    "data": {
      "touchad": 
         "touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fkb%3FonOff%3D1%26cd%3D1916%26cardIdx%3D896"
    }
  },
  "apns": {
    "headers": {
      "apns-priority": "10"
    },
    "payload": {
      "aps": {
        "alert": {
          "title": "터치애드",
          "subtitle": "광고시청하면 포인트리 적립!!!",
          "body": "출퇴근시간 지하철 통신비 절약되는 리브광고 많이 이용해주세요."
        },
        "category": "EVENT_INVITATION"
      },
      "touchad": 
		"touchad%3A%2F%2Ft.ta.runcomm.co.kr%2Fsrv%2Fadvertise%2Fmobile%2Fselect%2Fkb%3FonOff%3D1%26cd%3D1916%26cardIdx%3D896"
    },
    "fcm_options": {
      "image": "https://ta.runcomm.co.kr/html/img/profile00.png"
    }
  },
  "tokens": [
    "f3T_OObOQX-yo4J3y5bjcG:APA91bGzI2k8Fiz41ivql0ZV10hXLJz7w11Ne5Nf9IiZ1FymlJcGi-QRzv2lg3k46AYKamx-va2dyzj7m6TJTfCSTzTuPA7chomgSO_7PIh4LjsJ33SP7pDUoPvlGOeiM6oi5YXLiGvL"
  ]
}
~~~




* 매체사 앱의 Push Receiver에서는 Push 수신 후 data Property에서 파싱한 String값을 터치애드SDK의 onMessageReceived함수로 전달후 return합니다.
* 매체사 앱에서 푸시 수신시 Push Receiver 예시입니다.
~~~
class FcmListenerService : FirebaseMessagingService() {  

   override fun onMessageReceived(remoteMessage:RemoteMessage) {  

      val mbrId : String = 멤버십 카드번호
      
      if(mbrId.isNullOrEmpty()){
          Toast.makeText(this, R.string.empty_mbr_id, Toast.LENGTH_SHORT).show()
      }else{
          val pushData : String? = remoteMessage.data["touchad"]
          if(pushData.isNullOrEmpty()){
              Toast.makeText(this, R.string.null_data, Toast.LENGTH_SHORT).show()
          }else{
              val decodePushData = URLDecoder.decode(pushData, "UTF-8")
              
              //1차푸시
              decodePushData?.let {
                  //decodePushData json.touchad 값존재할경우 함수 호출
                  val json = JSONObject(decodePushData)
                  val touchad = json.getString("touchad")
                  
                  if touchad != null
                  {
                     TouchAdPlatform.openKBAdvertise(this, mbrId, it)
                  }
              }
              
              //2차 푸시
              decodePushData?.let {
                  //decodePushData json.pointree 값존재하고  json.platformId == "KBF" 일경우 함수 호출
                  val json = JSONObject(decodePushData)
                  val pointree = json.getString("pointree")
                  val platformId = json.getString("platformId")
                  
                  if pointree != null && platformId == "KBF"
                  {
                    TouchAdPlatform.openKBEarningResult(this, mbrId, it)
                  }
              }
          }
      }  
   }  
}
~~~





## Sample 프로젝트

* 프로젝트명 : and_TouchAd
* 패키지명 : kr.co.touchad
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.
