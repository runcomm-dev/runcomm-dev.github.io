# TouchAd 소개
* 매체사 제휴서비스인 런컴광고전송플랫폼 서비스(이하, 터치애드) 안드로이드 앱용 SDK 설치에 관한 내용입니다.
* 터치애드 서비스는 광고 포인트 적립 플랫폼입니다.
* 제휴앱 회원이 터치애드를 통해 광고를 이용할 경우 포인트 적립이 됩니다.
* 사용자가 광고 이용 후 포인트 적립이 이루어집니다.

  ## 광고전송 SDK 주요기능
  * 모바일 웹 광고 화면 : 일반적인 광고적립 앱 서비스(예:캐시슬라이드)에 사용되는 광고목록화면
  * SNS 회원가입 : 모바일 웹을 통한 SNS 회원가입
  * SKT 통합 회원 가입 : SKT 통신사를 이용하는 회원 대상
  * FCM Push 기능 : 적립, 통신비 차감, 전면광고 결과화면

   ## 매체사 앱 SDK 연동을 위한 업무진행 절차
  * 광고SDK에서 발급하는 platform id값을 획득하여 앱프로젝트 코딩에 사용.
  * 광고SDK 라이브러리(.aar) 앱프로젝트 Import.
  * 추가코딩 (SDK초기화, 전면광고적용, FCM리시버)
  * FCM전송 Public Web API(POST방식) 제작후 광고SDK담당자에게 전달
  * 테스트용 apk파일 광고SDK 담당자에게 전달
  * 광고SDK 기능테스트 (가입,적립,차감,FCM)
  * 상세한 기술적 내용은 아래 TouchAd SDK구성, TouchAd SDK 설치 가이드를 참고하시기 바랍니다.







# TouchAd SDK 구성

* 터치애드 SDK에 대한 설명입니다.
* 터치애드 SDK For 매체사 앱은 안드로이드 스튜디오(4.0.1)으로 개발되었습니다.
* SDK결과물은 확장자 aar 형태로 별도 제공됩니다.
* 안드로이드 minSdkVersion : 21 , targetSdkVersion : 28, compileSdkVerison : 29 (으)로 빌드되었습니다.



### SDK build.gradle(app)

* 터치애드 SDK는 <b>http라이브러리(retrofit2, OkHttp3), 이미지 로딩 라이브러리(glide), FCM, 이벤트로그(firebase), 자바비동기 이벤트 기반 라이브러리(rxjava2), 카메라 라이브러리(camerax), Firebase 디버깅 라이브러리(crashlytics)</b>를 사용합니다.
* buildTypes에 proguard에 대한 debug와 release에 따른 동작, stacktrace에 대한 예외처리를 위한 buildConfigField를 설정하였습니다.
* buildTypes에 consumerProguardFile을 사용하여 라이브러리 프로젝트에서 난독화 규칙을 제공하여 매체사 앱 프로젝트에 자동으로 규칙이 적용 됩니다.
* 아래는 SDK에 실제 적용된 내용입니다.

~~~
apply plugin: 'com.android.library'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'

android {

    compileSdkVersion 29

    defaultConfig {
        minSdkVersion 21
        targetSdkVersion 28
        versionCode 1016
        versionName "0.0.1"
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"

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
    flavorDimensions "flavors"
    productFlavors{
        dev{
            dimension "flavors"
        }
        product{
            dimension "flavors"
        }
        //라이브러리 모듈에 Flavor를 추가했을 경우 아래 옵션으로 기본 Flavor를 설정해 주어야
        // Android Studio에서 Run이 정상 동작을 한다.
        //Error:All flavors must now belong to a named flavor dimension. The flavor 'flavor_name' is not assigned to a flavor dimension.
        defaultPublishConfig "devDebug"
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

dependencies {
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.72"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation 'com.squareup.retrofit2:retrofit:2.5.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.5.0'
    implementation 'com.squareup.okhttp3:okhttp:3.12.0'
    implementation 'com.squareup.okhttp3:logging-interceptor:3.12.0'
    androidTestImplementation 'androidx.test.ext:junit:1.1.1'
    compile files('libs/card.io-5.5.1.jar')

    def camerax_version = "1.0.0-beta03"
    // CameraX core library using camera2 implementation
    implementation "androidx.camera:camera-camera2:$camerax_version"
    // CameraX Lifecycle Library
    implementation "androidx.camera:camera-lifecycle:$camerax_version"
    // CameraX View class
    implementation "androidx.camera:camera-view:1.0.0-alpha10"
    implementation 'com.google.firebase:firebase-messaging:20.2.1'

    implementation 'com.google.firebase:firebase-core:17.4.3'
    implementation "androidx.viewpager2:viewpager2:1.0.0"
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
    implementation 'com.github.bumptech.glide:glide:4.8.0'
    implementation 'com.google.firebase:firebase-crashlytics:17.2.2'
    implementation 'com.google.firebase:firebase-analytics:17.2.0'
    implementation 'com.makeramen:roundedimageview:2.3.0'
}
~~~



### AndroidManifest.xml

* SDK 내부에 사용되는 resource 아이디는 APK와 충돌하지 않게 네이밍 합니다.
* 아래에 권한설정 내용에 주석으로 권한 내용과 권한레벨을 작성하였으니 참고하시면 됩니다.
* 권한 내용 중 CAMERA, WRITE_EXTERNAL_STORAGE, SYSTEM_ALERT_WINDOW와 같이 **위험, 특별권한 레벨**은 인트로 화면이 끝나고 가이드 화면이 시작될 때 샘플 프로젝트 GuideActivity의 checkRequiredPermission() 함수 내에서 카메라, 외장메모리사용, 다른 앱 위에 그리기 권한을 **사용자**가 모두 수락할 경우 앱을 계속 진행하게 되며 위 세개의 권한 중 하나라도 거부를 할 경우 ‘모든 권한을 수락하셔야 합니다.’ 라는 토스트가 띄워 주면서 앱을 종료하게 됩니다.
* 아래는 소스코드 레벨에서 권한을 설정한 내용으로 위험, 특별 권한 레벨 설정 시 참고하시면 됩니다.
~~~
private fun checkRequiredPermission() {
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.M) {
            return
        }
        permissionHelper = PermissionHelper(arrayOf(Manifest.permission.WRITE_EXTERNAL_STORAGE, Manifest.permission.CAMERA), context)
        if (permissionHelper!!.checkPermissionInApp()) {
            val canDrawble = Settings.canDrawOverlays(this@GuideActivity)
            if (!canDrawble) {
                val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:$packageName"))
                startActivityForResult(intent, REQ_CODE_OVERLAY_PERMISSION)
            }
            return
        }
        permissionHelper!!.requestPermission(0, object : PermissionCallback {
            @RequiresApi(api = Build.VERSION_CODES.M)
            override fun onPermissionResult(permissions: Array<String>, grantResults: IntArray?) {
                mPermissions = permissions as Array<String?>
                mGrantResults = grantResults
                if (grantResults!!.isNotEmpty()) {
                    for (i in grantResults.indices) {
                        if (grantResults[i] != PackageManager.PERMISSION_GRANTED) {
                            Toast.makeText(applicationContext, "모든 권한을 수락하셔야합니다.", Toast.LENGTH_LONG).show()
                            finish()
                            break
                        }
                    }
                }
                val canDrawble = Settings.canDrawOverlays(this@GuideActivity)
                if (!canDrawble && permissionHelper!!.checkPermissionInApp()) {
                    val intent = Intent(Settings.ACTION_MANAGE_OVERLAY_PERMISSION, Uri.parse("package:$packageName"))
                    startActivityForResult(intent, REQ_CODE_OVERLAY_PERMISSION)
                }
            }
        })
    }
~~~




* 참조용으로 SDK 내에 설정된 내용입니다.
~~~
   <?xml version="1.0" encoding="utf-8"?>
   <manifest xmlns:android="http://schemas.android.com/apk/res/android"
       package="kr.co.touchad">
   
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
   
       <!--외장메모리 사용 권한 // 권한 레벨 : 위험-->
       <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
   
       <uses-feature android:name="android.hardware.camera.any" />
   
       <!--카메라 사용 권한 // 권한 레벨 : 위험-->
       <uses-permission android:name="android.permission.CAMERA" />
   
       <!--Android 10(API 29) 이상에서 전체화면 활동 실행 권한 // 권한 레벨 : 일반-->
       <uses-permission android:name="android.permission.USE_FULL_SCREEN_INTENT" />
   
       <!--다른 앱 위에 그리기 권한 // 권한 레벨 : 특별-->
       <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
   
       <application
           android:icon="@mipmap/tc_ic_launcher"
           android:label="@string/app_name"
           android:roundIcon="@mipmap/tc_ic_launcher_round"
           android:supportsRtl="true"
           android:allowBackup="false"
           android:usesCleartextTraffic="true"
           android:theme="@style/TouchAdTheme">
   
           <!--CPI 광고 처리를 위한 서비스 -->
           <service
               android:name="kr.co.touchad.ui.service.TouchAdService"
               android:enabled="true">
               <intent-filter>
                   <action android:name="kr.co.touchad.ui.service" />
                   <category android:name="android.intent.category.DEFAULT" />
               </intent-filter>
           </service>
   
           <!-- 웹뷰화면 -->
           <activity android:name="kr.co.touchad.ui.activity.webview.WebViewActivity"
               android:theme="@style/TouchAdTheme">
           </activity>
   
           <!-- 전체 광고 화면 -->
           <activity android:name="kr.co.touchad.ui.activity.advertise.AdFullActivity"
               android:theme="@style/TouchAdTheme">
           </activity>
   
           <!-- 카드등록 화면 -->
           <activity android:name="kr.co.touchad.ui.activity.card.CardRegisterActivity"
               android:theme="@style/TouchAdTheme"
               android:windowSoftInputMode="stateHidden">
           </activity>
   
           <!-- 카드 스캔 화면 -->
           <activity android:name="kr.co.touchad.ui.activity.card.CardScanActivity"
               android:theme="@style/TouchAdTheme"
               android:configChanges="orientation"
               android:screenOrientation="portrait">
           </activity>
   
           <activity android:name="io.card.payment.DataEntryActivity"/>
   
           <!-- 딥링크 게이트웨이 -->
           <activity android:name="kr.co.touchad.ui.activity.deeplink.SchemeActivity"
               android:launchMode="singleTask"
               android:theme="@style/Theme.TransparentTouchAd"
               android:noHistory="true">
               <intent-filter >
                   <action android:name="android.intent.action.VIEW" />
                   <category android:name="android.intent.category.DEFAULT" />
                   <category android:name="android.intent.category.BROWSABLE" />
                   <data android:scheme="touchad" android:host="touchad.co.kr" android:pathPrefix="/main" />
                   <data android:scheme="touchad" android:host="touchad.co.kr" android:pathPrefix="/main/advertise" />
                   <data android:scheme="touchad" android:host="touchad.co.kr" android:pathPrefix="/main/advertise_point_list" />
                   <data android:scheme="touchad" android:host="touchad.co.kr" android:pathPrefix="/main/card_register" />
               </intent-filter>
           </activity>
       </application>
   </manifest>
~~~



### proguard-rules.pro 파일

* jar형태로 구성된 라이브러리와 달리, **aar로 배포되는 터치애드 SDK는 난독화 규칙을 포함하여 배포할 수 있습니다.** proguard-rules.pro파일에 아래 내용이 추가되었으며 매체사 앱 측에서 별도로 **SDK에 대한 난독화 규칙을 추가하지 않습니다.**
* 추가로 retrofit2및 glide, stactrace오류보고에 대한 난독화 예외도 추가합니다.
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

* 터치애드는 CPI광고(Cost Per Install) 설치 체크를 위해 CPI광고 참여시 background service를 시작합니다.

* targetSdkVersion 26부터 적용되는 background service 실행 제한정책으로 해당 service가 background -> foreground로 변경되었습니다.

* Foreground Service를 시작하기 위해서 앱은 해당 서비스가 백그라운드로 실행되고 있다는것을 사용자에게 알려야하며, 해당 알림은 알람창의 notification 형태로 노출됩니다.

* 터치애드 foreground service 시작시 아래와 같은 notification이 알림창에 노출됩니다.

     ![그리기이(가) 표시된 사진  자동 생성된 설명](https://lh5.googleusercontent.com/GHExixOTfH_DnR__bCkyYx8roKyzjRN1jSPWfysLy00C44CT7P8OuGUyyOZgYyy8_ZF6Gc6Z-p0GmDs3aBCOzHiw_XKK_ZdRmVe7VEMFXRLG3X9Lebb4vRJ9rcDa6k3ztx5k7yBF)







#  TouchAd SDK 설치 가이드

* 정상적인 제휴서비스를 위한 터치애드SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 **touchad-sdk-1.0.0.aar** 파일을 프로젝트의 libs폴더에 넣어줍니다.



## build.gradle 설정 

  1. **build.gradle(project)파일수정**
     * Project build.gradle의 내용은 수정하지 않습니다.
     
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
apply plugin: 'com.google.firebase.crashlytics'


android {  
compileSdkVersion 29  

defaultConfig {  
applicationId "kr.co.touchad"  
minSdkVersion 21  
targetSdkVersion 28  
versionCode 1016  
versionName "0.0.1"  
testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"  
}  
buildTypes {  
release {

minifyEnabled true

proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'),

'proguard-rules.pro'

}

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
///////추가내용//////

repositories{  
flatDir{  
dirs 'libs'  
}  
}

////////////////////  
dependencies {  
implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:1.3.72"  
implementation 'androidx.appcompat:appcompat:1.1.0'  
implementation 'androidx.constraintlayout:constraintlayout:1.1.3'  
implementation 'com.squareup.retrofit2:retrofit:2.5.0'  
implementation 'com.squareup.retrofit2:converter-gson:2.5.0'  
implementation 'com.squareup.okhttp3:okhttp:3.12.0'  
implementation 'com.squareup.okhttp3:logging-interceptor:3.12.0'  
def camerax_version = "1.0.0-beta03"  
// CameraX core library using camera2 implementation  
implementation "androidx.camera:camera-camera2:$camerax_version"  
// CameraX Lifecycle Library  
implementation "androidx.camera:camera-lifecycle:$camerax_version"  
// CameraX View class  
implementation "androidx.camera:camera-view:1.0.0-alpha10"  
implementation 'com.google.firebase:firebase-messaging:20.2.1'  
implementation 'com.google.firebase:firebase-core:17.4.3'  
implementation "androidx.viewpager2:viewpager2:1.0.0"  
implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'  
implementation 'com.github.bumptech.glide:glide:4.8.0'

implementation 'com.google.firebase:firebase-crashlytics:17.2.2'

///////////////////추가내용/////////////////////////////

implementation name: ’touchad-sdk-1.0.0’, ext: ’arr’

///////////////////////////////////////////////////////  
implementation 'com.google.firebase:firebase-analytics:17.2.0'  
testImplementation 'junit:junit:4.12'  
androidTestImplementation 'androidx.test.ext:junit:1.1.1'  
androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'  
implementation 'com.makeramen:roundedimageview:2.3.0'  
}
~~~



## 터치애드 플랫폼 클래스 함수

   - 기능을 모듈화하여 Static 함수형태로 호출합니다.
   -  아래 간략한 설명입니다.

~~~
object TouchAdPlatform {  

/**
* 터치애드 초기화 함수. 메인 액티비티 진입시 최초 호출한다.
* @param context, options
*/
fun initialize(context: Context?, options: Bundle?)

/**
* 푸시 수신시 호출하는 함수.
* @param context, data
*/
fun  onMessageReceived(context: Context, data: String?)

/**
* 터치애드 전면광고 오픈
* @param context, link
*/
fun  openTouchAdAdvertise(context: Context)

/**
* 터치애드 카드등록 오픈
* @param context, link
*/
fun  openTouchAdCard(context: Context)

/**
* 터치애드 충전소 화면 시작
* @param context, link
*/
fun  startTouchAdWebview(context: Context) 

}

/**
* 터치애드 초기화 클래스, (4)터치애드 초기화에 설명
* @param mbrId, platformId
*/
class Builder(private val mbrId: String, private val platformId: String) {

fun setLargeIcon(largeIcon: Int)
fun setSmallIcon(smallIcon: Int)
fun setNotiToken(notiToken: String)
fun setAdsId(adsId: String)
fun create()

}
~~~



## 터치애드 초기화

* **앱의 메인화면 진입시 터치애드의 초기화 함수를 호출합니다.**

* **TouchAdPlatform.Builder 클래스를 이용하여 필수 파라미터 및 옵셔널한 파라미터 정보를 설정하여 초기화 함수를 호출합니다.**

*  **(필수) TouchAdPlatform.Builder(mbr_id, platform_id) :**
**mbr_id** -> SDK의 회원 인덱스 혹은 회원 구분값.  
**platform_id** -> 터치애드 관리자가 발급한 플랫폼 키값.

* **주의** : ***mbr_id, platform_id값은 필수 파라미터이며 둘중에 하나라도 빈값일경우  IllegalStateException이 발생합니다.***

* **(옵션) setLargeIcon** : 터치애드 관련 푸시 수신시 앱에서 알림을 생성하지 않고, SDK에서 푸시 데이터를 전달 받아 notification을 생성합니다. notification생성시 사용할 largeIcon의 리소스 아이디를 설정합니다. 설정하지 않을시 AndroidManifes application tag의 icon을 largeIcon으로 사용합니다.

* **(옵션) setSmallIcon** : notification 생성시 사용할 smallIcon의 리소스 아이디를 설정합니다. 설정하지 않을 시 AndroidManifest의 icon을 smallIcon으로 사용합니다. (***application tag의 icon이 존재하지 않을경우 터치애드 자체 smallIcon 사용***)

* **(옵션) setAdsId** : 등록된 광고를 전송하기 위한 아이디값을 설정해줍니다.  Default 값은 “”(Empty String)입니다.

* **(옵션) setNotiToken** : notification 생성시 사용할 토큰을 설정합니다.  Default 값은 “”(Empty String)입니다.

* **아래는 터치애드 초기화 함수 호출 예시입니다.**
~~~
class MainActivity : AppCompatActivity {

  override fun onCreate(savedInstanceState: Bundle?) {
    ……
    var mbrId = BasePreference.getValue(this, BasePreference.MBR_ID, "")!!
    var adsId = BasePreference.getValue(this, BasePreference.ADS_ID, "")!!
    var notiToken = BasePreference.getValue(this, BasePreference.NOTI_TOKEN, "")!!
    val builder = mbrId?.let { TouchAdPlatform.Builder(it, Constants.PLATFORM_ID) };

    builder?.setLargeIcon(R.mipmap.tc_ic_launcher)
    builder?.setSmallIcon(R.mipmap.tc_ic_launcher)
    builder?.setAdsId(adsId)
    builder?.setNotiToken(notiToken)
    
    TouchAdPlatform.initialize(this, builder?.create())
    ……
}
~~~



##  터치애드 전면광고 화면 시작

* 매체사 앱내에서 특정 트리거(결재완료, 기타 이벤트) 시점에 터치애드의 전면광고 화면을  띄울 때 호출합니다.

* 회원가입된 유저일경우 전면광고 화면으로 이동합니다.

* 비회원일경우 광고플랫폼 등록에 따른 약관 동의 화면이 활성화 됩니다.

* ※ 호출전 메인화면에서 initialize함수를 이용하여 **터치애드 초기화**가 이루어져야합니다.

* 아래는 터치애드 전면광고 시작함수 호출 예시입니다.
~~~
TouchAdPlatform.openTouchAdAdvertise(context);
~~~



##  터치애드 광고 플랫폼 회원 처리 시작

* 매체사 앱 내에서 터치애드 충전소를 선택하여 진입하게 되면, 터치애드 회원가입 여부에 따라  약관동의 확인을 거치거나 충전소 리스트로 바로 진입합니다.
* **(중요)광고 플랫폼의 광고 참여는 카드등록이 되어야 참여를 할 수 있습니다.**
* 아래는 터치애드 충전소 화면 시작함수 호출 예시입니다.
~~~
TouchAdPlatform.startTouchAdWebview(context)
~~~



##  터치애드 카드 등록

* 카드등록을 하기 전에 카드를 카메라에 스캔하여 자동으로 입력하는 기능이 들어있습니다.(**현재 양각 카드만 스캔이 가능하며 양각이외의 카드 스캔기능이 추가될 수 있습니다.**)
* 광고 리스트 화면 우측상단의 메뉴버튼을 누르면 MyPage로 이동하게 되며 카드등록에 진입하여 **카드스캔(또는 직접입력) 후 등록**을 하면 광고참여가 가능하게 됩니다.
~~~
TouchAdPlatform.startTouchAdCard(context)
~~~



##  터치애드 푸시 수신 시

* 매체사 앱 Push Receiver에서 터치애드 관련 Push 수신 시, Data Property에 존재하는 터치애드 관련 데이터를 Parsing하여 SDK에 전달합니다.
*  푸시관련에 대한 세부 내용은 아래 FCM 관련 부분을 참고바랍니다.



##  FCM 전송

* 터치애드는 푸시 송신 시 앱 제작사에서 제공한 Public API를 이용하여 데이터를 전달하고 제작사 서버에서 앱으로 Push(FCM)을 전송하는 형태입니다.(협의중)
* Public API를 개발하신 후 광고 SDK 담당자에게 전달바랍니다.
* 요청 데이터 형식(key : touchad, value : 문자열)
~~~
"touchad": "{\"title\":\"TOUCHAD\",\"msg\":\"ADVERTISE\",\"touchad\":\"touchad://ta.runcomm.co.kr/srv/advertise/mobile/select?onOff=1&cd=1916&cardIdx=190\"}"
    }
~~~

* API를 통해 POST된 데이터를 FCM 데이터의 구성요소 중 data 프로퍼티에 담아서 FCM 전송 바랍니다. (* 변경 가능성 있습니다.)

* FCM 전송 포맷 예시
~~~
{
  "android": {
    "priority": "high",
    "data": {
      "touchad": "{\"title\":\"TOUCHAD\",\"msg\":\"ADVERTISE\",\"touchad\":\"touchad://ta.runcomm.co.kr/srv/advertise/mobile/select?onOff=1&cd=1916&cardIdx=190\"}"
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
          "subtitle": "광고시청하면 리브포인트 적립!!!",
          "body": "출퇴근시간 지하철 통신비 절약되는 리브광고 많이 이용해주세요."
        },
        "category": "EVENT_INVITATION"
      },
      "touchad": "touchad://ta.runcomm.co.kr/srv/advertise/mobile/select?onOff=1&cd=1916&cardIdx=190"
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




* 매체사 앱의 Push Receiver에서는 Push 수신 후 data Property에서 파싱한 Json String값을 터치애드SDK의 onMessageReceived함수로 전달후 return합니다.
* 매체사 앱에서 푸시 수신시 Push Receiver 예시입니다.
~~~
class FcmListenerService : FirebaseMessagingService() {  

   override fun onMessageReceived(remoteMessage:RemoteMessage) {  

      remoteMessage ?: return
      
      val pushData = remoteMessage.data["touchad"]
      
      pushData?.let { it:String
         TouchAdPlatform.onMessageReceived(this, it)  
      }  
   }  
}
~~~





## Sample 프로젝트

* 프로젝트명 : TouchAd_Sample_Android
* 패키지명 : kr.co.touchad
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.
