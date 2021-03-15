#  TouchAd SDK  for SKT 설치 가이드

* 정상적인 제휴서비스를 위한 터치애드SDK 설치과정을 설명합니다.
* 샘플 프로젝트를 참조하면 좀 더 쉽게 설치 가능합니다.
* 제공한 TouchadSDK.framework 폴더를 프로젝트 소스폴더내 적절히 위치시켜 줍니다.
* 앱프로젝트 target > general > Frameworks,Libraries, and Embedded Content 에서 add files 에서 TouchadSDK.framework폴더를 선택합니다.
* Frameworks,Libraries, and Embedded Content 메뉴에서 TouchadSDK.framework의 Embed 옵션을 ‘Embed & Sign’ 선택합니다.


## CocoaPods 설정
1. **Podfile 파일수정**
* SDK에서 사용하는 cocoapod 라이브러리 입니다.
* 프로젝트 Podfile 에 아래내용을 추가합니다.
```
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '9.0'
use_frameworks!

target 'TouchadSDK' do

  pod 'SnapKit', '~> 4.0.0'
  pod 'Alamofire', '~> 4.8.2'
  pod 'ObjectMapper'
  pod 'JWTDecode', '~> 2.4'
  
end
```

## 권한 설정
1. **카메라**
* 교통카드 카메라 인식기능에 카메라 사용권한이 필요합니다.
* 사용자가 권한을 거부하는 경우 카메라 기능이 동작하지 않습니다.
* 앱프로젝트 info.plist 에 아래내용을 추가합니다.

| Key | Type | Value |
|---|---|---|
| Information Property List|Dictionary|(1 item)|
| Privacy - Camera Usage Description|String|교통카드 사진촬영을 위해 필요합니다.|

* 앱프로젝트에서 카메라 권한을 직접 설정할 경우 아래와 같은 메서드를 작성하여 사용합니다.

```
func requestCameraPermission(){
        AVCaptureDevice.requestAccess(for: .video, completionHandler: { (granted: Bool) in
            if granted {
                print("Camera: 권한 허용")
            } else {
                print("Camera: 권한 거부")
            }
        })
    }
```

## 터치애드 플랫폼 클래스 함수

- 주요기능을 모듈화하여 Static 함수형태로 호출합니다.
- 아래 간략한 설명입니다.
```
public class TASDKManager: NSObject {
  
/**
* 터치애드 초기화 함수. 메인 액티비티 진입시 최초 호출한다.
* @param mbrId: 매체사 회원관리번호 (필수)
* @param platformId: 매체사 플랫폼 고정값  (필수)
*/
    func initialize(_ mbrId : String, platformId : String)
    
/**
* 터치애드 전면광고 오픈
* @param userInfo: apns custom data
*/
    func openAdvertise(_ userInfo: [AnyHashable : Any])

/**
* 터치애드 충전소 화면 시작
*/    
    func startTouchAdWebview()

}
```

## 터치애드 초기화

* **앱의 메인화면 진입시 터치애드의 초기화 함수를 호출합니다.**

* **TASDKManager.initialize 함수를 이용하여 필수 파라미터 및 옵셔널한 파라미터 정보를 설정하여 초기화 함수를 호출합니다.**

**(필수) mbr_id -> 매체사 회원 관리번호**
**(필수) platform_id -> 터치애드 관리자가 발급한 매체사 구분값**

* **주의** : ***pushToken 경우 매체사와의 협의결과에 따라 전달 여부 결정*** 

* **아래는 터치애드 초기화 함수 호출 예시입니다.**
```
    //MARK: - App Launched
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
        NetworkRequest.requestLogin(nil, snsId: snsId, sns: sns, id: id , pwd: pwd, pwdDecrytYn: "N", success: { (response) in
            if let user = response.data?[0] as User? {
                TAGlobalManager.userInfo = user.toJSON()
                
                //FCM토큰을 전달하지 않는 경우 터치애드SDK 초기화
                TASDKManager.initialize(String(user.usrIdx!), platformId:TAConstants.PLATFORM_ID)
                                        
            }
        })
    }
```

## 터치애드 전면광고 화면 시작

* 매체사 앱내 특정 이벤트(푸시수신, 결제완료) 발생시점에 터치애드의 전면광고 화면을 띄울 경우 호출합니다.

* 회원가입된 터치애드 유저일경우 전면광고 화면으로 이동합니다.

* 비회원일경우 터치애드 등록에 따른 약관 동의 화면이 활성화 됩니다.

* ※ 호출전 메인화면에서 initialize함수를 이용하여 터치애드 초기화가 이루어져야합니다.

* 아래는 터치애드 전면광고 시작함수 호출 예시입니다.
```
TASDKManager.openAdvertise(userInfo)
```

## 터치애드 광고 플랫폼 회원처리 시작

* 매체사 앱 내에서 터치애드 충전소를 선택하여 진입하게 되면, 터치애드 회원가입 여부에 따라 약관동의 확인을 거치거나 충전소 리스트로 바로 진입합니다

* 아래는 터치애드 충전소 화면 시작함수 호출 예시입니다.
```
TASDKManager.startTouchAdWebview()
```

## 터치애드 카드 등록

* 카드등록을 하기 전에 카드를 카메라에 스캔하여 자동으로 입력하는 기능이 들어있습니다.(양각 카드 경우 인식률이 떨어질수 있습니다.)

* 광고 플랫폼의 광고 참여는 카드등록이 되어야 참여를 할 수 있습니다.

* 광고 리스트 화면 우측상단의 메뉴버튼을 누르면 MyPage로 이동하게 되며 카드등록에 진입하여 카드스캔(또는 직접입력) 후 등록을 하면 광고참여가 가능하게 됩니다.

## FCM 전송

* 터치애드는 푸시 송신 시 앱 제작사에서 제공한 Public API를 이용하여 데이터를 전달하고 제작사 서버에서 앱으로 Push(FCM)을 전송하는 형태입니다.(협의중)
* Public API를 개발하신 후 광고 SDK 담당자에게 전달바랍니다.
* Form파라미터(**필수**)

| 파라미터 | 내용 |
|---|---|
| `touchad`|JSON 문자열|

* API를 통해 Post된 데이터를 FCM데이터 구성요소 중 data와 payload 프로퍼티에 담아서 FCM전송 바랍니다.(※ 변경 가능성 있습니다.)

* FCM 전송 포맷 예시
```
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
```

## 빌드시  주의사항

* 애플 앱스토어 혹은 TestFlight 를 통한 앱배포시에는 x86_64 아키텍쳐 빌드가 제외된 SDK 로 빌드하여야 합니다.
* armv7, arm64  빌드 SDK :  폴더/ios_touchAd/배포용/TouchadSDK.framework
* XCode 에뮬레이터를 이용한 앱 개발시에는 x86_64 아키텍쳐 빌드가 포함된 SDK 로 빌드하여야 합니다.
* armv7, arm64, x86_64 빌드 SDK : 폴더/ios_touchAd/개발용/TouchadSDK.framework

## Sample 프로젝트

* 프로젝트명 : TouchAd_IOS
* 위 설명한 모든 내용이 실제 코딩이 되어 있습니다.
* 실제 SDK 설치 시 참조하면 도움이 될 것입니다.

