# Google Maps

## API 키 발급

### 키 생성
1. 구글 개발자 콘솔(console.cloud.google.com) 접속
2. 프로젝트 생성
3. API 및 서비스 > API 라이브러리
4. Maps SDK for Android 사용 설정
5. 키 및 사용자 인증 정보 > 사용자 인증 정보 만들기 > API 키

### 키 제한사항

1. 어플리케이션 제한사항 설정 > Android 앱
2. Android 제한 사항
	- 패키지 이름 입력
	- SHA-1 인증서 디지털 지문 등록
	
### 안드로이드 앱 인증서 지문 출력

디버그용 인증서 정보는 gradle signingReport 명령으로 확인 가능


## 키 등록

### 매니페스트 설정

매니페스트 파일에 API 키 등록

```
<application>
	<meta-data
        android:name="com.google.android.geo.API_KEY"
        android:value="발급받은 API 키" />
</application>
```

### Secrets Gradle Plugin

API 키를 형상관리 저장소에 등록하지 않기 위해 Secrets Gradle Plugin 을 사용할 수 있다.

1. 프로젝트 build.gradle 파일에 플러그인 설정

```
plugins {
    id("com.google.android.libraries.mapsplatform.secrets-gradle-plugin") version "2.0.1" apply false
}
```

2. 모듈 build.gradle

```
plugins {
    id("com.google.android.libraries.mapsplatform.secrets-gradle-plugin")
}
```

3. local.properties 파일에 KEY 등록

```
# Google Map Key
MAPS_API_KEY=발급받은 API KEY
```

4. 매니페스트 수정
```
<application>
	<meta-data
        android:name="com.google.android.geo.API_KEY"
        android:value="${MAPS_API_KEY}" />
</application>
```

## 위치 권한

1. ACCESS_COARSE_LOCATION : 와이파이 & 모바일 위치 접근, 위치 사용을 위해 기본적으로 필요한 권한

2. ACCESS_FINE_LOCATION : 위성, 정밀한 위치 접근  

3. ACCESS_BACKGROUND_LOCATION : 백그라우드 상태에서 위치 접근

```
<manifest ... >
  <!-- Always include this permission -->
  <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

  <!-- Include only if your app benefits from precise location access. -->
  <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
  
  <!-- Required only when requesting background location access on
       Android 10 (API level 29) and higher. -->
  <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />  
</manifest>
```


안드로이드 10 부터는 포그라운드 상태에서 기기의 위치 정보에 접근하기 위해 서비스 android:foregroundServiceType 지정 필요

```
<!-- Required for Android 10 (API level 29) and higher. -->
<service
    android:name="MyNavigationService"
    android:foregroundServiceType="location" ... >
    <!-- Any inner elements would go here. -->
</service>
```


## 권한 요청

```
val locationPermissionRequest = registerForActivityResult(
        ActivityResultContracts.RequestMultiplePermissions()
    ) { permissions ->
        when {
            permissions.getOrDefault(Manifest.permission.ACCESS_FINE_LOCATION, false) -> {
                // Precise location access granted.
            }
            permissions.getOrDefault(Manifest.permission.ACCESS_COARSE_LOCATION, false) -> {
                // Only approximate location access granted.
            } else -> {
                // No location access granted.
            }
        }
    }

// ...

// Before you perform the actual permission request, check whether your app
// already has the permissions, and whether your app needs to show a permission
// rationale dialog. For more details, see Request permissions.
locationPermissionRequest.launch(arrayOf(
    Manifest.permission.ACCESS_FINE_LOCATION,
    Manifest.permission.ACCESS_COARSE_LOCATION))
```

## FusedLocationProviderClient

FusedLocationProviderClient 는 Google Play 서비스으 일부이기 때문에 GoogleApiClient 를 별도로 설정하거나 관리할 필요가 없다.


> GoogleApiClient 와 FusedLocationProviderApi 조합은 이제 deprecated 되었다.

### Setup

```
dependencies {
    implementation("com.google.android.gms:play-services-location:21.0.1")
}

```

```
private lateinit var fusedLocationClient: FusedLocationProviderClient

override fun onCreate(savedInstanceState: Bundle?) {
    // ...

    fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)
}
```

### Usage

- 기기의 현재 위치를 확인 : lastLocation, currentLocation
- 기기의 위치 변화를 추적 : requestLocationUpdates

### lastLocation

빠르고, 배터리도 덜 쓰지만, 위치 정보 정확하지 않을 수 있다. 

```
fusedLocationClient.lastLocation
        .addOnSuccessListener { location : Location? ->
            // Got last known location. In some rare situations this can be null.
        }
```

### currentLocation

최신의 신뢰할 수 있는 위치 정보를 얻을 수 있지만, 호출 시 위치 계산이 발생한다.

권장되는 방법

```
        val request = CurrentLocationRequest.Builder()
            .setPriority(Priority.PRIORITY_HIGH_ACCURACY)
            .setGranularity(Granularity.GRANULARITY_PERMISSION_LEVEL)
            .setDurationMillis(5000L)
            .setMaxUpdateAgeMillis(10_000L)
            .build()

        fusedLocationClient.getCurrentLocation(request, null)
            .addOnSuccessListener { location ->
                addCurrentPositionMarker(location)
            }
			
```

### requestLocationUpdates

locationCallback 을 사용하여 위치 정보의 주기적 업데이트 처리 가능.

onResume() 과 onPause() 이벤트에서 위치 정보 요청을 제어해야 한다.

```
private lateinit var locationCallback: LocationCallback

override fun onCreate(savedInstanceState: Bundle?) {
	locationCallback = object : LocationCallback() {
		override fun onLocationResult(result: LocationResult) {
			val location = result.lastLocation
			location?.let {
				Log.d("MapsActivity", "location update : ${it}")
				addCurrentPositionMarker(it)
			}
		}
	}
}
```

```
    override fun onResume() {
        super.onResume()
		
        val request = LocationRequest.Builder(Priority.PRIORITY_HIGH_ACCURACY, 10_000L).build()
        fusedLocationClient.requestLocationUpdates(request, locationCallback, Looper.getMainLooper())
    }
```	

```
    override fun onPause() {
        super.onPause()
        fusedLocationClient.removeLocationUpdates(locationCallback)
    }
```

## 지도에 그리기

### Marker

```
    fun addCurrentPositionMarker(location: Location) {
        val latLng = LatLng(location.latitude, location.longitude)
        val marker = MarkerOptions().apply {
            icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_AZURE))
            position(latLng)
            title("현재 위치")
        }
        val position = CameraPosition.Builder()
            .target(latLng)
            .zoom(16f)
            .build()

        mMap.addMarker(marker)
        mMap.animateCamera(CameraUpdateFactory.newCameraPosition(position))
    }

```

## See Also

- https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderApi

- https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderClient

- https://developer.android.com/training/location/request-updates
