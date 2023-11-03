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

* KB 버전 쓱쌓 SDK에 대한 설명입니다.
* 쓱쌓 SDK For KB 리브메이트 앱은 안드로이드 스튜디오(4.0.1)으로 개발되었습니다.
* SDK 결과물은 확장자 aar 형태로 별도 제공됩니다.
* 안드로이드 minSdkVersion : 21 , targetSdkVersion : 33, compileSdkVersion : 33 (으)로 빌드되었습니다.



### SDK build.gradle(app)

* 쓱쌓 SDK는 <b>http라이브러리(retrofit2, OkHttp3), 자바비동기 이벤트 기반 라이브러리(rxjava2)</b>를 사용합니다.
* buildTypes안에 proguard에 대한 debug와 release에 따른 동작, stacktrace에 대한 예외처리를 위한 buildConfigField를 설정하였습니다.
* buildTypes에 consumerProguard File을 사용하여 라이브러리 프로젝트에서 난독화 규칙을 제공하여 매체사 앱 프로젝트에 자동으로 규칙이 적용 됩니다.
* 아래는 SDK 프로젝트 build.gradle(app)에 실제 적용된 내용입니다.

~~~
plugins {
    id 'com.android.library'
    id 'org.jetbrains.kotlin.android'
}

android {
    namespace 'kr.co.touchad.sdk'
    compileSdkVersion 33

    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 33
        versionCode 1022
        versionName "2.3"
        multiDexEnabled true

    }

    buildFeatures {
        viewBinding = true
        buildConfig = true
    }

    buildTypes {
        debug {
            minifyEnabled false
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

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_11
        targetCompatibility JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = '11'
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
    implementation 'androidx.activity:activity:1.3.0-alpha08'
}
~~~



### AndroidManifest.xml

* SDK 내부에 사용되는 resource 아이디는 APK와 충돌하지 않게 네이밍 합니다.
* 아래에 권한설정 내용에 주석으로 권한 내용과 권한레벨을 작성하였으니 참고하시면 됩니다.
* 권한 내용 중 **위험 레벨 권한**인 READ_EXTERNAL_STORAGE는 적립문의 화면 내에서 사용하는 파일첨부 기능을 사용하기 위해 추가되었습니다.(20220311 업데이트)
* 전화관련 정보 읽기 권한인 READ_PHONE_STATE는 API LEVEL 29까지만 적용되어 API LEVEL 30 부터 전면광고 화면에서 전화상태 체크를 하지 않습니다.
* Android 12 업데이트 이후 구글 스토어 정책 변경으로 광고아이디 권한이 추가되었습니다. 아래 상세내용 주소를 첨부합니다.
* 광고아이디 권한 상세 내용 : https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient.Info
* Android 13 부터 저장소 권한 세분화 정책이 적용되어 이미지 읽기를 사용할 경우 READ_EXTERNAL_STORAGE 대신 READ_MEDIA_IMAGES를 사용해야 합니다.(20231103 업데이트)
* 아래는 소스코드 레벨에서 권한을 설정한 내용으로 위험, 특별 권한 레벨 설정 예시입니다.
~~~
private fun checkRequiredPermission() {
    if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
            if (permission == Manifest.permission.READ_EXTERNAL_STORAGE) {
                startGalleryPage()
            }
            return
        }

        permissionHelper = PermissionHelper(arrayOf(permission), context)

        if (permissionHelper!!.checkPermissionInApp()) {
            if (permission == Manifest.permission.READ_EXTERNAL_STORAGE || permission == Manifest.permission.READ_MEDIA_IMAGES)
            {
                startGalleryPage()
            }
            else if (permission == Manifest.permission.READ_PHONE_STATE)
            {
                if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                    val canDrawble = Settings.canDrawOverlays(context)
                    if (!canDrawble) {
                        val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:$packageName"))

                        resultOverlay.launch(intent)
                    }
                }
            }
            return
        }

        permissionHelper!!.requestPermission(0, object : PermissionHelper.PermissionCallback {
            override fun onPermissionResult(permissions: Array<String>, grantResults: IntArray?) {
                if (grantResults!!.isNotEmpty()) {
                    var isGranted : Boolean = true
                    for (i in grantResults.indices) {
                        if (grantResults[i] != PackageManager.PERMISSION_GRANTED) {
                            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R && permissions[i] == Manifest.permission.READ_PHONE_STATE)
                            {
                                continue
                            }
                            else
                            {
                                isGranted = false
                                break
                            }
                        }
                    }

                    if (isGranted)
                    {
                        if (permission == Manifest.permission.READ_EXTERNAL_STORAGE || permission == Manifest.permission.READ_MEDIA_IMAGES)
                        {
                            startGalleryPage()
                        }
                        else if (permission == Manifest.permission.READ_PHONE_STATE)
                        {
                            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.Q) {
                                val canDrawble = Settings.canDrawOverlays(context)
                                if (!canDrawble) {
                                    val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:$packageName"))
                                    resultOverlay.launch(intent)
                                }
                            }
                        }
                    }
                    else
                    {
                        if (permission == Manifest.permission.READ_EXTERNAL_STORAGE || permission == Manifest.permission.READ_MEDIA_IMAGES)
                        {
                            if (touchAdWebView!!.mFilePathCallback != null) {
                                //파일을 한번 오픈했으면 mFilePathCallback 를 초기화를 해줘야함
                                // -- 그렇지 않으면 다시 파일 오픈 시 열리지 않는 경우 발생
                                touchAdWebView!!.mFilePathCallback!!.onReceiveValue(null)
                                touchAdWebView!!.mFilePathCallback = null
                            }
                        }
                        Toast.makeText(applicationContext, "모든 권한을 수락하셔야 기능을 사용하실 수 있습니다.", Toast.LENGTH_SHORT).show()
                    }
                }
            }
        })
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

    <!--전화관련 정보 읽기 권한 // 권한 레벨 : 위험-->
    <uses-permission android:name="android.permission.READ_PHONE_STATE" android:maxSdkVersion="29"/>

    <!--광고아이디 얻기 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="com.google.android.gms.permission.AD_ID" />

    <!--저장소 읽기 권한 // 권한 레벨 : 위험-->
    <uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

    <!--안드로이드 13 이상부터 저장소 권한 세분화로 이미지 읽기를 할 때 사용하는 권한 // 권한 레벨 : 위험-->
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
        android:theme="@style/TouchAdTheme"
        android:requestLegacyExternalStorage="true">

        <!--CPI 광고 처리를 위한 서비스 -->
        <service
            android:name="kr.co.touchad.sdk.ui.service.TouchAdService"
            android:enabled="true"
            android:exported="false"
            android:permission="android.permission.FOREGROUND_SERVICE"
            android:protectionLevel="signature">
        </service>

        <!-- 웹뷰화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.webview.WebViewActivity"
            android:theme="@style/TouchAdTheme"
            android:exported="false">
        </activity>

        <!-- 전면 광고 화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.advertise.AdFullActivity"
            android:theme="@style/TouchAdTheme"
            android:exported="false">
        </activity>
        
    </application>
</manifest>
~~~



### proguard-rules.pro 파일

* jar형태로 구성된 라이브러리와 달리, **aar로 배포되는 쓱쌓 SDK는 난독화 규칙을 포함하여 배포할 수 있습니다.** proguard-rules.pro 파일에 아래 내용이 추가되었으며 매체사 앱 측에서 별도로 **SDK에 대한 난독화 규칙을 추가하지 않습니다.**
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

* 쓱쌓은 메인 테마로 AppCompat.Light.NoActionBar를 사용하며 테마 네이밍이 겹치지 않도록 주의합니다.

* 쓱쌓 메인 테마명 : name = TouchAdTheme



### Foreground Service

* 쓱쌓은 CPI 광고(Cost Per Install) 설치 체크를 위해 CPI 광고 참여시 background service를 시작합니다.

* targetSdkVersion 26부터 적용되는 background service 실행 제한정책으로 해당 service가 background -> foreground로 변경되었습니다.

* Foreground Service를 시작하기 위해서 앱은 해당 서비스가 백그라운드로 실행되고 있다는것을 사용자에게 알려야하며, 해당 알림은 알림창의 notification 형태로 노출됩니다.

* 쓱쌓 foreground service 시작시 아래와 같은 notification이 알림창에 노출됩니다.

     ![그리기이(가) 표시된 사진  자동 생성된 설명](https://user-images.githubusercontent.com/25914626/124223205-3f37aa00-db3e-11eb-812f-9c7d4c6ed49d.png)



#  쓱쌓 SDK For KB 설치 가이드

* 정상적인 제휴서비스를 위한 쓱쌓 SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 **touchad-sdk-2.3.aar** 파일을 프로젝트의 libs 폴더에 넣어줍니다.



## build.gradle 설정 

  1. **build.gradle(project)파일수정**
     *      * 광고Id를 가져와 쓱쌓 광고참여를 하기 위해 plugins에 com.google.gms.google-services를 추가합니다.
            * google-services 사용에 필요한 파일인 google-services.json파일은 샘플 프로젝트 내 gradle 파일과 같은 레벨에서 찾을 수 있습니다.
            * 아래는 실제 작성된 예시입니다.
~~~
plugins {
    id 'com.android.application' version '7.4.1' apply false
    id 'com.android.library' version '7.4.1' apply false
    id 'org.jetbrains.kotlin.android' version '1.6.20' apply false
    id 'com.google.gms.google-services' version '4.3.8' apply false
}
~~~

  2. **build.gradle(app)파일수정**
     *  아래 dependencies 영역내용을 추가합니다.
     *  build.gradle에  android{…}영역과 dependencies{…}사이에 repositories{flatDir{…}}을 추가합니다.
     *  dependencies 영역에 Implementation name: ’touchad-sdk-2.3’, ext: ’arr’를 추가합니다.
     *  중복된 내용은 생략 합니다.
~~~
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'com.google.gms.google-services'
}

android {
    namespace 'kb pay 패키지명'
    compileSdkVersion 33

    defaultConfig {
        applicationId "kb pay 패키지명"
        minSdkVersion 21
        targetSdkVersion 33
        versionCode 1022
        versionName "1.0"
        multiDexEnabled true
    }

    buildFeatures {
        viewBinding = true
    }

    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_11
        targetCompatibility JavaVersion.VERSION_11
    }

    kotlinOptions {
        jvmTarget = '11'
    }

    lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
}

dependencies {
    implementation 'androidx.core:core-ktx:1.8.0'
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'com.squareup.retrofit2:retrofit:2.5.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.5.0'
    implementation 'com.squareup.okhttp3:okhttp:3.12.13'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.12.13'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'

    implementation 'com.google.firebase:firebase-messaging:20.2.1'
    implementation 'com.google.firebase:firebase-core:17.4.3'
    implementation "androidx.viewpager2:viewpager2:1.0.0"
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'

    implementation files('libs/touchad-sdk-2.3.aar')

    implementation 'com.makeramen:roundedimageview:2.3.0'
    implementation 'com.auth0.android:jwtdecode:2.0.0'
    implementation 'androidx.multidex:multidex:2.0.0'
}
~~~



## 쓱쌓 플랫폼 클래스 함수

   - 기능을 모듈화하여 Static 함수형태로 호출합니다.
   -  아래 간략한 설명입니다.

~~~
object TouchAdPlatform {  

/**
* 쓱쌓 전면광고 화면 시작
*/
fun  openKBAdvertise(context: Context, cid: String, data: String)

/**
* 애드모아 화면 시작
*/
fun  openKBEarningMenu(context: Context, cid: String)

/**
* 적립문의 화면 시작
*/
fun  openKBInquiryMenu(context: Context, cid: String)

/**
* 이용안내 화면 시작
*/
fun  openKBUseInfoMenu(context: Context)

/**
* 공지사항 화면 시작
*/
fun  openKBNoticeMenu(context: Context)

/**
* 참여이력 화면 시작
*/
fun openKBApprlNoMenu(context: Context, cid: String)

/**
* 오늘의 쇼핑적립 화면 시작
*/
fun openKBShoppingMenu(context: Context, cid: String, cd: String?)

}
~~~

##  쓱쌓 전면광고 화면 시작

*  카드결제완료 후 푸시 수신하고 이때 쓱쌓의 전면광고 화면을 띄울 경우 호출합니다.
*  광고회원가입된 유저일경우 전면광고 화면으로 이동합니다.
*  cid = 고객관리번호(필수값)
*  data = 푸시데이터(필수값, FCM 항목 참고하시면 됩니다.)
* 쓱쌓 전면광고 시작함수 호출 시 data에 아래 예시와 같이 JSON String 전체를 전달하면 됩니다.

*  아래는 쓱쌓 전면광고 시작함수 호출 예시입니다.

~~~
val data: String = 
"{\"cid\":\"cd834b16c772a0755d133dd1322f2bc24e079f7b9640e71b064bf71fa55e7739\",
\"apprlNo\":\"12345678\",\"title\":\"LiivMate\",\"body\":\"쓱쌓에서 포인트가 도착했습니다.\",
\"custom-type\":\"touchad\",\"custom-body\":\"%7b%22touchad%22%3a%22touchad%3a%2f%2f2.ta.runcomm.co.kr
%2fsrv%2fadvertise%2fmobile%2fselect%2fkb%3fapprlNo%3d12345678%26cid%3d5a8d5abda44de97f7e0742f311f94b92da1813d1c51d1895adc73fea3c01d3d8%26adsIdx%3d15484%22%7d\"}"

TouchAdPlatform.openKBAdvertise(context, cid, data);
~~~

##  애드모아 화면 시작

*  KB 앱 내에서 참여적립 메뉴를 선택하면 약관동의를 거치고 애드모아 화면을 시작할때 호출합니다.
*  cid = 고객관리번호(필수값)
*  아래는 애드모아 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openKBEarningMenu(context, cid)
~~~

## 적립문의 화면 시작

*  KB 앱 내에서 적립문의를 선택 시 호출합니다.
*  cid = 고객관리번호(필수값)
*  아래는 적립문의 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openKBInquiryMenu(context, cid)
~~~

## 이용안내 화면 시작

*  KB 앱 내에서 이용안내를 선택 시 호출합니다.
*  아래는 이용안내 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openKBUseInfoMenu(context)
~~~

## 공지사항 화면 시작

*  KB 앱 내에서 공지사항을 선택 시 호출합니다.
*  아래는 공지사항 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openKBNoticeMenu(context)
~~~

## 참여이력 화면 시작

*  KB 앱 내에서 참여이력을 선택 시 호출합니다.
*  아래는 참여이력 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openKBApprlNoMenu(context, cid)
~~~

## 오늘의 쇼핑적립 화면 시작(매일매일 쇼핑적립 메인 화면 이동)

*  KB Pay Life 화면 내에서 런컴 CPS광고에 있는 '더보기' 선택 시 호출합니다.
*  cid = 고객관리번호(필수값)
*  cd = 광고 외부관리 코드(선택)
*  cd값에 null을 넣어야 매일매일 쇼핑적립 메인 화면으로 이동합니다.
*  아래는 오늘의 쇼핑적립 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openKBShoppingMenu(Context, cid, null)
~~~

## 오늘의 쇼핑적립 화면 시작(CPS 광고 터치 시 바로 광고 상세레이어 이동)

*  KB Pay Life 화면 내에서 런컴 CPS 광고 선택 시 호출합니다.
*  cid = 고객관리번호(필수값)
*  cd = 광고 외부관리 코드(선택)
*  cd값에 광고 외부관리 코드를 넣어야 매일매일 쇼핑적립 메인 화면 위에 나타나는 광고 상세 레이어로 이동합니다.
*  아래는 오늘의 쇼핑적립 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openKBShoppingMenu(Context, cid, cd(외부관리코드))
~~~

##  쓱쌓 푸시 수신 시

* 쓱쌓은 푸시 송신 시 KB 에서 제공한 Public API를 이용하여 Push(FCM)를 전송합니다.
* 매체사 앱 Push Receiver에서 쓱쌓 관련 Push 수신 시, Data Property에 존재하는 쓱쌓 관련 데이터를 Parsing하여 SDK에 전달합니다.
* 푸시관련에 대한 세부 내용은 아래 FCM 관련 부분을 참고바랍니다.



##  FCM 전송

* 쓱쌓은 푸시 송신 시 앱 제작사에서 제공한 Public API를 이용하여 데이터를 전달하고 제작사 서버에서 앱으로 Push(FCM)을 전송하는 형태입니다.(협의중)
* Public API를 개발하신 후 광고 SDK 담당자에게 전달바랍니다.
* 요청 데이터 형식(key : touchad, value : 문자열)
~~~
"{\"cid\":\"cd834b16c772a0755d133dd1322f2bc24e079f7b9640e71b064bf71fa55e7739\",
 \"apprlNo\":\"12345678\",\"title\":\"LiivMate\",\"body\":\"쓱쌓에서 포인트가 도착했습니다.\",
 \"custom-type\":\"touchad\",\"custom-body\":\"%7b%22touchad%22%3a%22touchad%3a%2f%2f2.ta.runcomm.co.kr
 %2fsrv%2fadvertise%2fmobile%2fselect%2fkb%3fapprlNo%3d12345678%26cid%3d5a8d5abda44de97f7e0742f311f94b92da1813d1c51d1895adc73fea3c01d3d8%26adsIdx%3d15484%22%7d\"}"
~~~

* API를 통해 POST된 데이터를 FCM 데이터의 구성요소 중 data 프로퍼티에 담아서 FCM 전송 바랍니다. (* 변경 가능성 있습니다.)

* FCM 전송 포맷 예시
~~~
{
  "android": {
    "priority": "high",
    "data": {
      "touchad": 
         "{\"cid\":\"cd834b16c772a0755d133dd1322f2bc24e079f7b9640e71b064bf71fa55e7739\",
          \"apprlNo\":\"12345678\",\"title\":\"LiivMate\",\"body\":\"쓱쌓에서 포인트가 도착했습니다.\",
          \"custom-type\":\"touchad\",\"custom-body\":\"%7b%22touchad%22%3a%22touchad%3a%2f%2f2.ta.runcomm.co.kr
          %2fsrv%2fadvertise%2fmobile%2fselect%2fkb%3fapprlNo%3d12345678%26cid%3d5a8d5abda44de97f7e0742f311f94b92da1813d1c51d1895adc73fea3c01d3d8%26adsIdx%3d15484%22%7d\"}"
    }
  },
  "apns": {
    "headers": {
      "apns-priority": "10"
    },
    "payload": {
      "aps": {
        "alert": {
          "title": "쓱쌓",
          "subtitle": "광고시청하면 포인트리 적립!!!",
          "body": "출퇴근시간 지하철 통신비 절약되는 리브광고 많이 이용해주세요."
        },
        "category": "EVENT_INVITATION"
      },
      "touchad": 
		"{\"cid\":\"cd834b16c772a0755d133dd1322f2bc24e079f7b9640e71b064bf71fa55e7739\",
         \"apprlNo\":\"12345678\",\"title\":\"LiivMate\",\"body\":\"쓱쌓에서 포인트가 도착했습니다.\",
         \"custom-type\":\"touchad\",\"custom-body\":\"%7b%22touchad%22%3a%22touchad%3a%2f%2f2.ta.runcomm.co.kr
         %2fsrv%2fadvertise%2fmobile%2fselect%2fkb%3fapprlNo%3d12345678%26cid%3d5a8d5abda44de97f7e0742f311f94b92da1813d1c51d1895adc73fea3c01d3d8%26adsIdx%3d15484%22%7d\"}"
    },
    "fcm_options": {
      "image": "https://2.ta.runcomm.co.kr/html/img/profile00.png"
    }
  },
  "tokens": [
    "f3T_OObOQX-yo4J3y5bjcG:APA91bGzI2k8Fiz41ivql0ZV10hXLJz7w11Ne5Nf9IiZ1FymlJcGi-QRzv2lg3k46AYKamx-va2dyzj7m6TJTfCSTzTuPA7chomgSO_7PIh4LjsJ33SP7pDUoPvlGOeiM6oi5YXLiGvL"
  ]
}
~~~




* 매체사 앱의 Push Receiver에서는 Push 수신 후 data Property에서 파싱한 String값을 쓱쌓SDK의 onMessageReceived함수로 전달후 return합니다.
* 매체사 앱에서 푸시 수신시 Push Receiver 예시입니다.
~~~
class FcmListenerService : FirebaseMessagingService() {  

   override fun onMessageReceived(remoteMessage:RemoteMessage) {  

      val pushData : String? = remoteMessage.data["touchad"]
      if(pushData.isNullOrEmpty()){
          Toast.makeText(this, R.string.null_data, Toast.LENGTH_SHORT).show()
      }else{
          TouchAdPlatform.openKBAdvertise(this, "고객관리번호", pushData)
      }  
   }  
}
~~~





## Sample 프로젝트

* 프로젝트명 : and_TouchAd
* 패키지명 : kr.co.touchad
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.
