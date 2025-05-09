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


# TouchAd SDK For DIGILOCA 구성

* DIGILOCA 버전 Touchad SDK에 대한 설명입니다.
* Touchad SDK For DIGILOCA 앱은 안드로이드 스튜디오(Hedgehog)로 개발되었습니다.
* SDK 결과물은 확장자 aar 형태로 별도 제공됩니다.
* 안드로이드 minSdkVersion : 23 , targetSdkVersion : 34, compileSdkVersion : 34 (으)로 빌드되었습니다.



### SDK build.gradle(app)

* Touchad SDK는 <b>http라이브러리(retrofit2, OkHttp3), 자바비동기 이벤트 기반 라이브러리(rxjava2)</b>를 사용합니다.
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
    compileSdk 34

    defaultConfig {
        minSdkVersion 23
        targetSdkVersion 34
        versionCode 1004
        versionName "1.4"
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
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = '1.8'
    }

    lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
}

dependencies {
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.9.0"
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
    implementation 'androidx.fragment:fragment-ktx:1.3.0-rc01'
}
~~~



### AndroidManifest.xml

* SDK 내부에 사용되는 resource 아이디는 APK와 충돌하지 않게 네이밍 합니다.
* 아래에 권한설정 내용에 주석으로 권한 내용과 권한레벨을 작성하였으니 참고하시면 됩니다.
* 전화관련 정보 읽기 권한인 READ_PHONE_STATE는 API LEVEL 29까지만 적용되어 API LEVEL 30 부터 전면광고 화면에서 전화상태 체크를 하지 않습니다.
* Android 12 업데이트 이후 구글 스토어 정책 변경으로 광고아이디 권한이 추가되었습니다. 아래 상세내용 주소를 첨부합니다.
* 광고아이디 권한 상세 내용 : https://developers.google.com/android/reference/com/google/android/gms/ads/identifier/AdvertisingIdClient.Info
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

    <!--Task 정보를 구하는 권한 // 권한레벨 : signatureOrSystem-->
    <uses-permission android:name="android.permission.GET_TASKS"/>

    <!--광고아이디 얻기 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="com.google.android.gms.permission.AD_ID" />

    <queries>
        <intent>
            <action android:name="android.intent.action.MAIN" />
            <category android:name="android.intent.category.LAUNCHER" />
        </intent>
    </queries>

    <application
        android:supportsRtl="true"
        android:allowBackup="false"
        android:usesCleartextTraffic="true"
        android:hardwareAccelerated="true"
        android:requestLegacyExternalStorage="true">

        <!-- 웹뷰화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.webview.WebViewActivity"
            android:theme="@style/TouchAdTheme"
            android:windowSoftInputMode="adjustResize"
            android:exported="false">
        </activity>

        <!-- 전체 광고 화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.advertise.AdFullActivity"
            android:theme="@style/TouchAdTheme"
            android:exported="false">
        </activity>

    </application>
</manifest>
~~~



### proguard-rules.pro 파일

* jar형태로 구성된 라이브러리와 달리, **aar로 배포되는 Touchad SDK는 난독화 규칙을 포함하여 배포할 수 있습니다.** proguard-rules.pro 파일에 아래 내용이 추가되었으며 매체사 앱 측에서 별도로 **SDK에 대한 난독화 규칙을 추가하지 않습니다.**
* 추가로 retrofit2및 stacktrace 오류보고에 대한 난독화 예외도 추가합니다.
* 아래 코드는 SDK에 추가된 Proguard-rules.pro에 대한 내용입니다.
~~~
-keep class kr.co.touchad.sdk.** {public *;}#패키지 하위 클래스 중 public 메소드만 난독화x
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

#  Touchad SDK For DIGILOCA 설치 가이드

* 정상적인 제휴서비스를 위한 Touchad SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 **touchad-sdk-1.4.aar** 파일을 프로젝트의 libs 폴더에 넣어줍니다.



## build.gradle 설정 

  1. **build.gradle(project)파일수정**
     *      * 광고Id를 가져와 Touchad 광고참여를 하기 위해 plugins에 com.google.gms.google-services를 추가합니다.
            * google-services 사용에 필요한 파일인 google-services.json파일은 샘플 프로젝트 내 gradle 파일과 같은 레벨에서 찾을 수 있습니다.
            * 아래는 실제 작성된 예시입니다.
~~~
plugins {
    id 'com.android.application' version '8.1.1' apply false
    id 'com.android.library' version '8.1.1' apply false
    id 'org.jetbrains.kotlin.android' version '1.9.0' apply false
    id 'com.google.gms.google-services' version '4.3.8' apply false
}
~~~

  2. **build.gradle(app)파일수정**
     *  아래 dependencies 영역내용을 추가합니다.
     *  dependencies 영역에 Implementation name: ’touchad-sdk-1.0’, ext: ’arr’를 추가합니다.
     *  중복된 내용은 생략 합니다.
~~~
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'com.google.gms.google-services'
}

android {
    namespace 'kr.co.touchad'
    compileSdk 34

    defaultConfig {
        applicationId "DIGILOCA 패키지명"
        minSdkVersion 23
        targetSdkVersion 34
        versionCode 1004
        versionName "1.4"
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
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }

    kotlinOptions {
        jvmTarget = '1.8'
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

    implementation files('libs/touchad-sdk-1.4.aar')

    implementation 'com.makeramen:roundedimageview:2.3.0'
    implementation 'com.auth0.android:jwtdecode:2.0.0'
    implementation 'androidx.multidex:multidex:2.0.0'
    implementation 'androidx.activity:activity:1.3.0-alpha08'
    implementation 'androidx.fragment:fragment-ktx:1.3.0-rc01'
}
~~~



## Touchad 플랫폼 클래스 함수

   - 기능을 모듈화하여 Static 함수형태로 호출합니다.
   -  아래 간략한 설명입니다.

~~~
object TouchAdPlatform {  

/**
* 하루세번머니 화면 시작
*/
fun openTodayEarningMenu(context: Context, isProd: Boolean, cid: String, gender : String, birthYear : String)

/**
* 라방보고머니 화면 시작
*/
fun openLaBangEarningMenu(context: Context, isProd: Boolean, cid: String, gender : String, birthYear : String)

}
~~~

##  하루세번머니 화면 시작

*  디지로카 앱 내에서 하루세번머니 메뉴를 선택하면 약관동의를 거치고 하루세번머니 화면을 시작할때 호출합니다.
*  isProd = 개발 / 상용 구분 값(true = 상용, false = 개발)
*  cid = 고객관리번호(필수값)
*  gender = 성별(ex: 남성 = M, 여성 = F)
*  birthYear = 출생년도(ex: 1992)
*  아래는 하루세번머니 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openTodayEarningMenu(context, isProd, cid, gender, birthYear)
~~~

##  라방보고머니 화면 시작

*  디지로카 앱 내에서 라방보고머니 메뉴를 선택하면 약관동의를 거치고 라방보고머니 화면을 시작할때 호출합니다.
*  isProd = 개발 / 상용 구분 값(true = 상용, false = 개발)
*  cid = 고객관리번호(필수값)
*  gender = 성별(ex: 남성 = M, 여성 = F)
*  birthYear = 출생년도(ex: 1992)
*  아래는 라방보고머니 화면 시작함수 호출 예시입니다.

~~~
TouchAdPlatform.openLaBangEarningMenu(context, isProd, cid, gender, birthYear)
~~~

## Sample 프로젝트

* 프로젝트명 : and_TouchAd
* 패키지명 : kr.co.touchad
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.
