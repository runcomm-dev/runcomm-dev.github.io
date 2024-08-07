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


# TouchAd SDK For NH멤버스 구성

* NH멤버스 버전 NH터치애드 SDK에 대한 설명입니다.
* NH터치애드 SDK For NH멤버스 앱은 안드로이드 스튜디오(4.2.1)으로 개발되었습니다.
* SDK 결과물은 확장자 aar 형태로 별도 제공됩니다.
* 안드로이드 minSdkVersion : 21 , targetSdkVersion : 34, compileSdkVersion : 34 (으)로 빌드되었습니다.



### SDK build.gradle(app)

* NH터치애드 SDK는 <b>http라이브러리(retrofit2, OkHttp3), 자바비동기 이벤트 기반 라이브러리(rxjava2)</b>를 사용합니다.
* buildTypes안에 proguard에 대한 debug와 release에 따른 동작, stacktrace에 대한 예외처리를 위한 buildConfigField를 설정하였습니다.
* buildTypes에 consumerProguard File을 사용하여 라이브러리 프로젝트에서 난독화 규칙을 제공하여 매체사 앱 프로젝트에 자동으로 규칙이 적용 됩니다.
* 아래는 SDK 프로젝트 build.gradle(app)에 실제 적용된 내용입니다.

~~~
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'

android {

    compileSdkVersion 34

    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 34
        versionCode 1005
        versionName "1.5"
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
    implementation 'com.squareup.okhttp3:okhttp:3.12.13'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.12.13'
    implementation 'com.google.firebase:firebase-core:17.4.3'
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
    implementation 'com.auth0.android:jwtdecode:2.0.0'
}
~~~



### AndroidManifest.xml

* SDK 내부에 사용되는 resource 아이디는 APK와 충돌하지 않게 네이밍 합니다.
* 아래에 권한설정 내용에 주석으로 권한 내용과 권한레벨을 작성하였으니 참고하시면 됩니다.
* 권한 내용 중 **위험 레벨 권한**인 READ_EXTERNAL_STORAGE는 적립문의 화면 내에서 사용하는 파일첨부 기능을 사용하기 위해 추가되었습니다.
* Android 12 업데이트 이후 구글 스토어 정책 변경으로 광고아이디 권한이 추가되었습니다. 아래 상세내용 주소를 첨부합니다.
* 광고아이디 권한 상세 내용 : https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient.Info
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

    <!--광고아이디 얻기 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="com.google.android.gms.permission.AD_ID" />

    <!--저장소 사용 권한 // 권한 레벨 : 위험-->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
    
    <!--Android 13 이상 저장소 사용 권한 // 권한 레벨 : 위험-->
    <uses-permission android:name="android.permission.READ_MEDIA_IMAGES"/>

    <queries>
        <intent>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent>
    </queries>

    <application
        android:icon="@mipmap/tc_ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/tc_ic_launcher_round"
        android:supportsRtl="true"
        android:allowBackup="false"
        android:usesCleartextTraffic="true"
        android:hardwareAccelerated="true"
        android:theme="@style/TouchAdTheme">

        <!-- 웹뷰화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.webview.WebViewActivity"
            android:theme="@style/TouchAdTheme">
        </activity>

        <!-- 전면 광고 화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.advertise.AdFullActivity"
            android:theme="@style/TouchAdTheme">
        </activity>
        
    </application>
</manifest>
~~~



### proguard-rules.pro 파일

* jar형태로 구성된 라이브러리와 달리, **aar로 배포되는 NH터치애드 SDK는 난독화 규칙을 포함하여 배포할 수 있습니다.** proguard-rules.pro 파일에 아래 내용이 추가되었으며 매체사 앱 측에서 별도로 **SDK에 대한 난독화 규칙을 추가하지 않습니다.**
* 추가로 retrofit2및 stacktrace 오류보고에 대한 난독화 예외도 추가합니다.
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

##---------------Begin: proguard configuration for Gson  ----------
# Gson uses generic type information stored in a class file when working with fields. Proguard
# removes such information by default, so configure it to keep all of it.
-keepattributes Signature

# For using GSON @Expose annotation
-keepattributes *Annotation*

# Gson specific classes
-dontwarn sun.misc.**
#-keep class com.google.gson.stream.** { *; }

# Application classes that will be serialized/deserialized over Gson
-keep class com.google.gson.examples.android.model.** { <fields>; }

# Prevent proguard from stripping interface information from TypeAdapter, TypeAdapterFactory,
# JsonSerializer, JsonDeserializer instances (so they can be used in @JsonAdapter)
-keep class * extends com.google.gson.TypeAdapter
-keep class * implements com.google.gson.TypeAdapterFactory
-keep class * implements com.google.gson.JsonSerializer
-keep class * implements com.google.gson.JsonDeserializer

# Prevent R8 from leaving Data object members always null
-keepclassmembers,allowobfuscation class * {
  @com.google.gson.annotations.SerializedName <fields>;
}

##---------------End: proguard configuration for Gson  ----------

-keepclassmembers enum * {
   public static **[] values();
   public static ** valueOf(java.lang.String);
}
~~~



### styles.xml 파일

* NH터치애드는 메인 테마로 AppCompat.Light.NoActionBar를 사용하며 테마 네이밍이 겹치지 않도록 주의합니다.

* NH터치애드 메인 테마명 : name = TouchAdTheme


#  NH터치애드 SDK For NH멤버스 설치 가이드

* 정상적인 제휴서비스를 위한 NH터치애드 SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 **touchad-sdk-nh-1.5.aar** 파일을 프로젝트의 libs 폴더에 넣어줍니다.



## build.gradle 설정 

  1. **build.gradle(project)파일수정**
     *      * 광고Id를 가져와 NH터치애드 광고참여를 하기 위해 아래 dependencies의 calsspath에 google-services를 추가합니다.
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
     *  dependencies 영역에 Implementation name: ’touchad-sdk-nh-1.5’, ext: ’arr’를 추가합니다.
     *  중복된 내용은 생략 합니다.
~~~
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'
apply plugin: 'com.google.gms.google-services'

android {
    compileSdkVersion 34

    defaultConfig {
        applicationId "kr.co.touchad"
        minSdkVersion 21
        targetSdkVersion 34
        versionCode 1005
        versionName "1.5"
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
    implementation 'com.squareup.okhttp3:okhttp:3.12.13'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.12.13'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'

    implementation 'com.google.firebase:firebase-messaging:20.2.1'
    implementation 'com.google.firebase:firebase-core:17.4.3'
    implementation "androidx.viewpager2:viewpager2:1.0.0"
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
    implementation 'com.github.bumptech.glide:glide:4.8.0'

    implementation name: 'touchad-sdk-nh-1.5', ext: 'aar'

    implementation 'com.makeramen:roundedimageview:2.3.0'
    implementation 'com.auth0.android:jwtdecode:2.0.0'
    implementation 'androidx.multidex:multidex:2.0.0'
}
~~~



## NH터치애드 플랫폼 클래스 함수

   - 기능을 모듈화하여 Static 함수형태로 호출합니다.
   - 아래 간략한 설명입니다.

~~~
object TouchAdPlatform {  

/**
* NH띠링 전면광고 화면 시작
*/
fun  openNHAdvertise(context: Context, isProd: Boolean, cid: String, data: String)

/**
* NH터치애드 화면 시작
*/
fun  openNHEarningMenu(context: Context, isProd: Boolean, cid: String)
 
}
~~~

##  NH띠링 전면광고 화면 시작

*  NH멤버스내 포인트 사용/적립 후 후 푸시 수신 후 터치시 NH띠링의 전면광고 화면을 띄울 경우 호출합니다.
*  광고회원가입된 유저일경우 전면광고 화면으로 이동합니다.
*  isProd = 개발 / 상용 도메인을 설정하는 Boolean 값(필수 값, true = 상용 도메인, false = 개발 도메인)
*  cid = 고객관리번호(필수값)
*  data = 푸시데이터(필수값, FCM 항목 참고하시면 됩니다.)
* NH띠링 전면광고 시작함수 호출 시 data에 아래 예시와 같이 JSON String 전체를 전달하면 됩니다.

*  아래는 NH띠링 전면광고 시작함수 호출 예시입니다.

~~~
val isProd: Boolean = true(상용 도메인) 또는 false(개발 도메인)

val data: String = 
"{\"cid\":\"3jmkTE4EYMVBAw/1SFdzUA==\",
\"apprlNo\":\"12345678\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",
\"custom-type\":\"touchad\",\"custom-body\":\"%7b%22touchad%22%3a%22touchad%3a%2f%2f1.ta.runcomm.co.kr
%2fsrv%2fadvertise%2fmobile%2fselect%2fnh%3fapprlNo%3d12345678%26cid%3d5a8d5abda44de97f7e0742f311f94b92da1813d1c51d1895adc73fea3c01d3d8%26adsIdx%3d15484%22%7d\",\"platformId\":\"NHJ\"}"

TouchAdPlatform.openNHAdvertise(context, isProd, cid, data);
~~~

##  NH터치애드 화면 시작

*  NH멤버스 앱 내에서 NH터치애드 메뉴를 선택하면 약관동의를 거치고 NH터치애드 화면을 시작할때 호출합니다.
*  isProd = 개발 / 상용 도메인을 설정하는 Boolean 값(필수 값, true = 상용 도메인, false = 개발 도메인)
*  cid = 고객관리번호(필수값)
*  아래는 NH터치애드 화면 시작함수 호출 예시입니다.

~~~
val isProd: Boolean = true(상용 도메인) 또는 false(개발 도메인)

TouchAdPlatform.openNHEarningMenu(context, isProd, cid)
~~~

##  NH띠링 푸시 수신 시

* NH띠링은 푸시 송신 시 NH멤버스 에서 제공한 Public API를 이용하여 Push(FCM)를 전송합니다.
* 매체사 앱 Push Receiver에서 NH띠링 관련 Push 수신 시, Data Property에 존재하는 NH띠링 관련 데이터를 Parsing하여 SDK에 전달합니다.
* 푸시관련에 대한 세부 내용은 아래 FCM 관련 부분을 참고바랍니다.



##  FCM 전송

* NH띠링은 푸시 송신 시 앱 제작사에서 제공한 Public API를 이용하여 데이터를 전달하고 제작사 서버에서 앱으로 Push(FCM)을 전송하는 형태입니다.(협의중)
* Public API를 개발하신 후 광고 SDK 담당자에게 전달바랍니다.
* 요청 데이터 형식(key : touchad, value : 문자열)
~~~
"{\"cid\":\"3jmkTE4EYMVBAw/1SFdzUA==\",
 \"apprlNo\":\"12345678\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",
 \"custom-type\":\"touchad\",\"custom-body\":\"%7b%22touchad%22%3a%22touchad%3a%2f%2f1.ta.runcomm.co.kr
 %2fsrv%2fadvertise%2fmobile%2fselect%2fnh%3fapprlNo%3d12345678%26cid%3d5a8d5abda44de97f7e0742f311f94b92da1813d1c51d1895adc73fea3c01d3d8%26adsIdx%3d15484%22%7d\",\"platformId\":\"NHJ\"}"
~~~

* API를 통해 POST된 데이터를 FCM 데이터의 구성요소 중 data 프로퍼티에 담아서 FCM 전송 바랍니다. (* 변경 가능성 있습니다.)

* FCM 전송 포맷 예시
~~~
개발 도메인 : t.ta.runcomm.co.kr
상용 도메인 : 1.ta.runcomm.co.kr
{
  "android": {
    "priority": "high",
    "data": {
      "touchad": 
         "{\"cid\":\"3jmkTE4EYMVBAw/1SFdzUA==\",
          \"apprlNo\":\"12345678\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",
          \"custom-type\":\"touchad\",\"custom-body\":\"%7b%22touchad%22%3a%22touchad%3a%2f%2f1.ta.runcomm.co.kr
          %2fsrv%2fadvertise%2fmobile%2fselect%2fnh%3fapprlNo%3d12345678%26cid%3d5a8d5abda44de97f7e0742f311f94b92da1813d1c51d1895adc73fea3c01d3d8%26adsIdx%3d15484%22%7d\",\"platformId\":\"NHJ\"}"
    }
  },
  "apns": {
    "headers": {
      "apns-priority": "10"
    },
    "payload": {
      "aps": {
        "alert": {
          "title": "NH띠링",
          "subtitle": "광고시청하면 포인트 적립!!!",
          "body": "출퇴근시간 지하철 통신비 절약되는 NH멤버스 광고 많이 이용해주세요."
        },
        "category": "EVENT_INVITATION"
      },
      "touchad": 
		"{\"cid\":\"3jmkTE4EYMVBAw/1SFdzUA==\",
         \"apprlNo\":\"12345678\",\"title\":\"NH멤버스\",\"body\":\"NH띠링에서 포인트가 도착했습니다.\",
         \"custom-type\":\"touchad\",\"custom-body\":\"%7b%22touchad%22%3a%22touchad%3a%2f%2f1.ta.runcomm.co.kr
         %2fsrv%2fadvertise%2fmobile%2fselect%2fnh%3fapprlNo%3d12345678%26cid%3d5a8d5abda44de97f7e0742f311f94b92da1813d1c51d1895adc73fea3c01d3d8%26adsIdx%3d15484%22%7d\"}"
    },
    "fcm_options": {
      "image": "https://1.ta.runcomm.co.kr/html/img/profile00.png"
    }
  },
  "tokens": [
    "f3T_OObOQX-yo4J3y5bjcG:APA91bGzI2k8Fiz41ivql0ZV10hXLJz7w11Ne5Nf9IiZ1FymlJcGi-QRzv2lg3k46AYKamx-va2dyzj7m6TJTfCSTzTuPA7chomgSO_7PIh4LjsJ33SP7pDUoPvlGOeiM6oi5YXLiGvL"
  ]
}
~~~




* 매체사 앱의 Push Receiver에서는 Push 수신 후 data Property에서 파싱한 String값을 NH터치애드 SDK의 onMessageReceived함수로 전달후 return합니다.
* 푸시 수신시 앱이 포그라운드 상태일 경우와 백그라운드 상태일 경우를 분기하셔야합니다.
* 포그라운드 상태일 경우 : NH띠링 전면광고 화면 노출
* 백그라운드 또는 종료 상태일 경우 : 인트로 후 NH멤버스 메인 화면 위에 NH띠링 전면광고 화면 노출
* 매체사 앱에서 푸시 수신시 Push Receiver 예시입니다.
~~~
class FcmListenerService : FirebaseMessagingService() {  

   override fun onMessageReceived(remoteMessage:RemoteMessage) {  

      val pushData : String? = remoteMessage.data["touchad"]

      //true = 상용 도메인, false = 개발 도메인
      val isProd : Boolean = true

      if(pushData.isNullOrEmpty()){
          Toast.makeText(this, R.string.null_data, Toast.LENGTH_SHORT).show()
      } else {
          if (isAppInBackground(this)) { 
             //백그라운드 또는 종료상태
             startNotificationFinishApp()

             //인트로 후 NH멤버스 메인 화면 위에 NH띠링 전면광고 SDK 함수 호출
             TouchAdPlatform.openNHAdvertise(this, isProd, "고객관리번호", pushData)
          } else { 
             //포그라운드 상태
             //전면광고 SDK 함수 호출
             TouchAdPlatform.openNHAdvertise(this, isProd, "고객관리번호", pushData)
          }
      }  
   }  
}

//백그라운드 또는 앱이 종료상태일 때 실행할 Notification 예시 코드입니다.
private fun startNotificationFinishApp()
    {
        createNotificationChannel(this, NotificationManagerCompat.IMPORTANCE_HIGH, false,
            getString(R.string.app_name), "App notification channel")

        val channelId = "$packageName-${getString(R.string.app_name)}"
        val title = "NH터치애드 푸시 테스트"
        val content = "푸시테스트입니다."

        //SplashActivity(인트로화면)에서 NH멤버스메인으로 이동한 후에 터치애드 SDK 전면광고 함수를 호출하시면 됩니다.
        val intent = Intent(baseContext, SplashActivity::class.java)
        intent.putExtra("nhTouchadPush", "nhTouchadPush")

        val fullScreenPendingIntent : PendingIntent
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.S)
        {
            fullScreenPendingIntent = PendingIntent.getActivity(baseContext, 0,
                intent, PendingIntent.FLAG_IMMUTABLE)
        }
        else
        {
            fullScreenPendingIntent = PendingIntent.getActivity(baseContext, 0,
                intent, PendingIntent.FLAG_UPDATE_CURRENT)
        }

        val builder = NotificationCompat.Builder(this, channelId)
        builder.setSmallIcon(R.mipmap.tc_ic_launcher)
        builder.setContentTitle(title)
        builder.setContentText(content)
        builder.setPriority(NotificationCompat.PRIORITY_HIGH)//builder.priority = NotificationCompat.PRIORITY_HIGH
        builder.setAutoCancel(true)
        builder.setFullScreenIntent(fullScreenPendingIntent, true)

        val notificationManager = NotificationManagerCompat.from(this)
        notificationManager.notify(1002, builder.build())
}

//앱의 현재 상태 체크 함수 예시 코드입니다.
private fun isAppInBackground(context: Context): Boolean {
        var isInBackground = true
        val am = context.getSystemService(ACTIVITY_SERVICE) as ActivityManager
        val runningProcesses = am.runningAppProcesses
        for (processInfo in runningProcesses) {
            if (processInfo.importance == RunningAppProcessInfo.IMPORTANCE_FOREGROUND) {
                for (activeProcess in processInfo.pkgList) {
                    if (activeProcess == context.packageName) {
                        isInBackground = false
                    }
                }
            }
        }
        return isInBackground
    }
~~~





## Sample 프로젝트

* 프로젝트명 : and_TouchAd
* 패키지명 : kr.co.touchad
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.
