# TouchAd 소개
* 매체사 제휴서비스인 런컴광고전송플랫폼 서비스(이하, 터치애드) 안드로이드 앱용 SDK 설치에 관한 내용입니다.
* 터치애드 서비스는 광고 포인트 적립 플랫폼입니다.
* 제휴앱 회원이 터치애드를 통해 광고를 이용할 경우 포인트 적립이 됩니다.
* 사용자가 광고 이용 후 포인트 적립이 이루어집니다.

  ## 광고전송 SDK 주요기능
    * 모바일 웹 광고 화면 : 일반적인 광고적립 앱 서비스(예:캐시슬라이드)에 사용되는 광고목록화면
    * OCB 통합 회원 가입 : OCB를 이용하는 회원 대상

  ## 매체사 앱 SDK 연동을 위한 업무진행 절차
    * 광고 SDK 라이브러리(.aar) 앱프로젝트 Import.
    * 추가코딩 (터치애드 화면 호출)
    * 테스트용 apk 파일 광고 SDK 담당자에게 전달
    * 광고 SDK 기능테스트 (가입, 해지, 적립)
    * 상세한 기술적 내용은 아래 TouchAd SDK 구성, TouchAd SDK 설치 가이드 항목을 참고하시기 바랍니다.


# TouchAd SDK For OCB 구성

* OCB 버전 터치애드 SDK에 대한 설명입니다.
* 터치애드 SDK For OCB 앱은 안드로이드 스튜디오(Hedgehog)로 개발되었습니다.
* SDK 결과물은 확장자 aar 형태로 별도 제공됩니다.
* 안드로이드 minSdkVersion : 26 , targetSdkVersion : 34, compileSdkVersion : 34 (으)로 빌드되었습니다.



### SDK build.gradle(app)

* 터치애드 SDK는 <b>http라이브러리(retrofit2, OkHttp3), 자바비동기 이벤트 기반 라이브러리(rxjava2)</b>를 사용합니다.
* buildTypes안에 proguard에 대한 debug와 release에 따른 동작, stacktrace에 대한 예외처리를 위한 buildConfigField를 설정하였습니다.
* buildTypes에 consumerProguard File을 사용하여 라이브러리 프로젝트에서 난독화 규칙을 제공하여 매체사 앱 프로젝트에 자동으로 규칙이 적용 됩니다.
* dependencies implementation 라이브러리 버전이 OCB에서 정의한 동일 라이브러리 버전과 상이할 경우 SDK 담당자에게 내용 전달바랍니다.
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
        minSdkVersion 26
        targetSdkVersion 34
        versionCode 1001
        versionName "1.1"
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
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = '17'
    }

    lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
}
dependencies {
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.9.0"
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.3'
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    implementation 'com.squareup.okhttp3:okhttp:4.10.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.9.2'
    implementation 'com.google.android.material:material:1.9.0'
    implementation 'com.google.firebase:firebase-analytics:21.5.0'
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
    implementation 'androidx.activity:activity:1.7.1'
    implementation 'com.auth0.android:jwtdecode:2.0.0'
    implementation 'androidx.fragment:fragment-ktx:1.5.4'
}
~~~



### AndroidManifest.xml

* SDK 내부에 사용되는 resource 아이디는 APK와 충돌하지 않게 네이밍 합니다.
* 아래에 권한설정 내용에 주석으로 권한 내용과 권한레벨을 작성하였으니 참고하시면 됩니다.
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

    <!--어플리케이션이 항상 켜져있도록 하는 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="android.permission.WAKE_LOCK" />

    <!--광고아이디 얻기 권한 // 권한 레벨 : 일반-->
    <uses-permission android:name="com.google.android.gms.permission.AD_ID" />

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
        android:requestLegacyExternalStorage="true">

        <!-- 웹뷰화면 -->
        <activity android:name="kr.co.touchad.sdk.ui.activity.webview.WebViewActivity"
            android:theme="@style/TouchAdTheme"
            android:screenOrientation="portrait"
            android:windowSoftInputMode="adjustResize"
            android:exported="false">
        </activity>
    </application>
</manifest>
~~~



### proguard-rules.pro 파일

* jar형태로 구성된 라이브러리와 달리, **aar로 배포되는 터치애드 SDK는 난독화 규칙을 포함하여 배포할 수 있습니다.** proguard-rules.pro 파일에 아래 내용이 추가되었으며 매체사 앱 측에서 별도로 **SDK에 대한 난독화 규칙을 추가하지 않습니다.**
* 추가로 retrofit2및 stactrace 오류보고에 대한 난독화 예외도 추가합니다.
* 아래 코드는 SDK에 추가된 Proguard-rules.pro에 대한 내용입니다.
~~~
-keep class kr.co.touchad.sdk.** {public *;}#패키지 하위 클래스 중 public 메소드만 난독화x
-keep class android.support.** { *; }
-keep class com.google.** { *; }
-keepparameternames#파라미터 이름을 난독화x

#소스 파일의 라인을 섞지 않는 옵션 ( 안하게되면 나중에 stacktrace보고 어느 line에서 오류가 난 것인지 확인 불가)
-keepattributes SourceFile,LineNumberTable

# retrofit2
-keepattributes Signature, InnerClasses, EnclosingMethod
-keepattributes RuntimeVisibleAnnotations, RuntimeVisibleParameterAnnotations
-keepattributes AnnotationDefault
-keepclassmembers,allowshrinking,allowobfuscation interface * {
    @retrofit2.http.* <methods>;
}
-dontwarn org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
-dontwarn javax.annotation.**
-dontwarn kotlin.Unit
-dontwarn retrofit2.KotlinExtensions
-dontwarn retrofit2.KotlinExtensions$*
-if interface * { @retrofit2.http.* <methods>; }
-keep,allowobfuscation interface <1>
-if interface * { @retrofit2.http.* <methods>; }
-keep,allowobfuscation interface * extends <1>
-keep,allowobfuscation,allowshrinking class kotlin.coroutines.Continuation
-if interface * { @retrofit2.http.* public *** *(...); }
-keep,allowoptimization,allowshrinking,allowobfuscation class <3>
-keep,allowobfuscation,allowshrinking class retrofit2.Response

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

##---------------Start: proguard configuration for jwtdecode(gradle plugin version 8.2 and higher)---------
-dontwarn com.auth0.android.jwt.JWT
-keep class com.auth0.android.jwt.JWTPayload {
    <fields>;
}
##---------------End: proguard configuration for jwtdecode(gradle plugin version 8.2 and higher)---------

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

##---------------Begin: proguard configuration for card.io  ----------

# ---- REQUIRED card.io CONFIG ----------------------------------------
# card.io is a native lib, so anything crossing JNI must not be changed

# Don't obfuscate DetectionInfo or public fields, since
# it is used by native methods
-keep class io.card.payment.DetectionInfo
-keepclassmembers class io.card.payment.DetectionInfo {
    public *;
}

-keep class io.card.payment.CreditCard
-keep class io.card.payment.CreditCard$1
-keepclassmembers class io.card.payment.CreditCard {
  *;
}

-keepclassmembers class io.card.payment.CardScanner {
  *** onEdgeUpdate(...);
}

# Don't mess with classes with native methods

-keepclasseswithmembers class * {
    native <methods>;
}

-keepclasseswithmembernames class * {
    native <methods>;
}

-keep public class io.card.payment.* {
    public protected *;
}

# required to suppress errors when building on android 22
-dontwarn io.card.payment.CardIOActivity

##---------------End: proguard configuration for card.io  ------------

-keepclassmembers enum * {
   public static **[] values();
   public static ** valueOf(java.lang.String);
}
~~~



### styles.xml 파일

* 터치애드는 메인 테마로 AppCompat.Light.NoActionBar를 사용하며 테마 네이밍이 겹치지 않도록 주의합니다.

* 터치애드 메인 테마명 : name = TouchAdTheme



#  TouchAd SDK For OCB 설치 가이드

* 정상적인 제휴서비스를 위한 터치애드 SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 **touchad-sdk-1.1.aar** 파일을 프로젝트의 libs 폴더에 넣어줍니다.



## build.gradle 설정

1. **build.gradle(project)파일수정**
    * 광고Id를 가져와 터치애드 광고참여를 하기 위해 아래 dependencies의 calsspath에 google-services를 추가합니다.
    * google-services 사용에 필요한 파일인 google-services.json파일은 샘플 프로젝트 내 gradle 파일과 같은 레벨에서 찾을 수 있습니다.
    * 아래는 실제 작성된 예시입니다.
~~~
// Top-level build file where you can add configuration options common to all sub-projects/modules.
plugins {
    id 'com.android.application' version '8.2.2' apply false
    id 'com.android.library' version '8.2.2' apply false
    id 'org.jetbrains.kotlin.android' version '1.8.10' apply false
    id 'com.google.gms.google-services' version '4.3.15' apply false
}
~~~

2. **build.gradle(app)파일수정**
    *  아래 dependencies 영역내용을 추가합니다.
    *  dependencies 영역에 implementation files('libs/touchad-sdk-1.1.aar')를 추가합니다.          
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
        applicationId "kr.co.touchad"
        minSdkVersion 26
        targetSdkVersion 34
        versionCode 1001
        versionName "1.1"
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
        debug {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'

        }
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_17
        targetCompatibility JavaVersion.VERSION_17
    }

    kotlinOptions {
        jvmTarget = '17'
    }

    lintOptions {
        checkReleaseBuilds false
        // Or, if you prefer, you can continue to check for errors in release builds,
        // but continue the build even when errors are found:
        abortOnError false
    }
}

dependencies {
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk8:1.9.0"
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0'
    implementation 'com.squareup.okhttp3:okhttp:4.10.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:4.9.2'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.3'
    implementation 'com.google.android.material:material:1.9.0'

    implementation 'com.google.firebase:firebase-analytics:21.5.0'
    implementation "androidx.viewpager2:viewpager2:1.0.0"
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.1'
    implementation 'com.auth0.android:jwtdecode:2.0.0'

    implementation files('libs/touchad-sdk-1.1.aar')

    implementation 'com.makeramen:roundedimageview:2.3.0'
    implementation 'androidx.multidex:multidex:2.0.0'
    implementation 'androidx.activity:activity:1.7.1'
    implementation 'androidx.fragment:fragment-ktx:1.5.4'
}
~~~

## Android 12 버전 업데이트로 인한 AndroidManifest 설정 안내
1. **android:exported 속성 추가**
    * 이 속성은 SDK에도 반영이 되어있습니다.
    * 앱이 Android 12 이상을 타겟팅하고 인텐트 필터를 사용하는 액티비티나 서비스, broadcast receiver를 포함할 시 명시적으로 선언해야합니다.
    * 명시적으로 선언을 하지 않을 경우 Android 12 이상을 실행하는 기기에 앱을 설치할 수 없습니다.
    * 앱 구성요소에 LAUNCHER 카테고리가 포함된 경우 android:exported를 true로 설정합니다.
    * 아래 링크 주소는 안드로이드 Developers 공식 관련 내용입니다.
    * https://developer.android.com/about/versions/12/behavior-changes-12?hl=ko
    * https://developer.android.com/guide/topics/manifest/activity-element?hl=ko#exported

## 터치애드 플랫폼 클래스 함수

- 기능을 모듈화하여 Static 함수형태로 호출합니다.
-  아래 간략한 설명입니다.

~~~
object TouchAdPlatform {  

/**
* 꽝없는 랜덤 포인트 화면 시작
*/
fun  openTodayEarningMenu(context: Context, isProd: Boolean, userId: String, gender:String?, birthYear: String?)

/**
* 포인트 더받기 화면 시작
*/
fun  openEarningMenu(context: Context, isProd: Boolean, userId: String, gender:String?, birthYear: String?)

}
~~~


##  꽝없는 랜덤 포인트 화면 시작

* OCB 앱 내 출석체크 화면에서 '꽝없는 랜덤 포인트' 버튼을 터치시 호출합니다.
* isProd = 개발 / 상용 도메인을 설정하는 Boolean 값(필수, true = 상용 도메인, false = 개발 도메인)
* userId = 고객식별번호(필수)
* gender = 성별(남자 : M, 여자 : F, 기타 : Z)
* birthYear = 출생년도(ex : 1996)

* 아래는 꽝없는 랜덤 포인트 화면 시작함수 예시입니다.

~~~
TouchAdPlatform.openTodayEarningMenu(context, isProd, userId, gender, birthYear)
~~~

##  포인트 더받기 화면 시작

* OCB 앱 내 출석체크 화면에서 '포인트 더받기' 버튼을 터치시 호출합니다.
* isProd = 개발 / 상용 도메인을 설정하는 Boolean 값(필수, true = 상용 도메인, false = 개발 도메인)
* userId = 고객식별번호(필수)
* gender = 성별(남자 : M, 여자 : F, 기타 : Z)
* birthYear = 출생년도(ex : 1996)

* 아래는 포인트 더받기 화면 시작함수 예시입니다.

~~~
TouchAdPlatform.openEarningMenu(context, isProd, userId, gender, birthYear)
~~~


## Sample 프로젝트

* 프로젝트명 : and_TouchAd
* 패키지명 : kr.co.touchad
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.
