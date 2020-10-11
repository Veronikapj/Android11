### **스코프 스토리지**

Android 10 도입> 앱과 사용자 데이터 보호와 file clutter를 줄이기 위해 설계

Android 11 발전 > 피드백 기반으로 업그레이드

 

### 자주하는 질문

공개포럼에 대한 토론을 기반으로 FAQ 생성

#### Scoped Storage를 통해 앱이 File API를 사용하여 파일 경로로 파일에 액세스 할 수 있습니까?

- 미디어 경로에 직접 액세스 하는 라이브러리, 코드가 있으므로, Android 11에서 외부 스토리지 읽기 권한이있는 앱은 범위가 지정된 스토리지 환경에서 **파일 경로로 파일에 액세스 할 수 있습니다.** 
  Android 10 기기에서는 android:requestLegacyExternalStorage 매니페스트 속성을 설정하여 옵션을 꺼야만한다. Android 버전에서 연속성을 유지하려면 앱이 Android 10 이상을 대상으로하는 경우 옵트 꺼야한다. 
- 자세한 내용은 [범위가 지정된 스토리지 모범 사례](https://developer.android.com/training/data-storage/use-cases#access-file-paths) 를 참조하십시오.

------

#### 파일 경로 액세스 성능은 Media Store API와 어떻게 다릅니까?

- 성능은 실제 사용 사례에 따라 다릅니다. 
  비디오 재생과 같은 **순차적 읽기**의 경우 파일 경로 액세스는 Media Store와 **비슷한 성능**을 제공합니다. 
  **임의 읽기 및 쓰기**의 경우 **파일 경로 사용이 최대 두 배 느릴 수** 있습니다. 
  가장 빠르고 일관된 읽기 및 쓰기를 위해서는 Media Store API를 권장합니다.

------

#### 내 앱은 공유 스토리지에 광범위하게 액세스해야합니다. Storage Access Framework가 유일한 옵션입니까?

- SAF (Storage Access Framework)는 사용자가 디렉토리 및 파일에 대한 액세스 권한을 부여 할 수있는 **옵션 중 하나**입니다. 그러나 루트 및 Android / data 디렉토리와 같은 특정 디렉토리 에는 액세스 제한이 있습니다. 
  Android / data 및 Android / obb 디렉토리를 제외한 외부 저장소의 모든 파일에 액세스 할 수 있는 MANAGE_EXTERNAL_STORAGE 권한을 추가했습니다 . 
- 예정된 정책 발표에서 Google Play는이 권한의 사용에 대한 추가 지침을 제공 할 것입니다.

------

#### 어떤 카테고리의 앱이 MANAGE_EXTERNAL_STORAGE권한을 요청해야 합니까?

- MANAGE_EXTERNAL_STORAGE 권한은 기기의 파일에 광범위하게 액세스 해야하지만 범위가 지정된 스토리지 모범 사례를 사용하여 효율적으로 수행 할 수없는 핵심 사용 사례가있는 앱을 위한 것입니다. 
  가능한 모든 사용 사례를 열거하는 것은 실용적이지 않지만 일부 사용 사례에는 **파일 관리자, 백업 및 복원, 안티 바이러스 앱 또는 생산성 파일 편집 앱**이 포함됩니다.

------

#### Storage Access Framework를 사용하려면 Google Play 정책 승인이 필요합니까?

- Storage Access Framework는 Android 4.4부터 플랫폼에 있습니다. Storage Access Framework를 통해 파일에 액세스하면 사용자가 파일을 선택하는 데 관여하고 사용자 권한이 필요하지 않으므로 더 나은 제어가 가능합니다. 
- 사용법과 관련된 Google Play 정책이 없습니다.

------

#### Android 10과 비교하여 Android 11에서 Storage Access Framework 사용에 대한 추가 제한 사항이 있습니까?

- Android 11 (API 레벨 30)을 대상으로하고 Storage Access Framework를 사용하는 앱은 더 이상 **SD 카드의 루트 디렉토리 및 다운로드 디렉토리와 같은 디렉토리에 대한 액세스 권한을 부여 할 수 없습니다**. 대상 SDK에 관계없이 Android 11의 Storage Access Framework를 사용하여 **Android/data 및 Android/obb 디렉토리에 액세스 할 수 없습니다.** 

------

#### 앱이 범위 스토리지 변경 사항을 테스트하는 방법

- 앱은 호환성 플래그*를 통해 직접 파일 경로 액세스 또는 Media Store API와 관련된 범위가 지정된 스토리지 동작을 테스트 할 수 있습니다 . Storage Access Framework를 사용하여 특정 경로에 액세스 하기 위한 제한 사항을 테스트하는 RESTRICT_STORAGE_ACCESS_FRAMEWORK 플래그 도 있습니다 .

*호환성 플래그

- DEFAULT_SCOPED_STORAGE (enabled for all apps by default)
- FORCE_ENABLE_SCOPED_STORAGE (disabled for all apps by default)

------

#### 범위가 지정된 저장소의 앱은 파일을 앱별 데이터 디렉토리에 쓰는 것으로 제한됩니까?

- 범위가 지정된 저장소에서 앱은 **미디어 파일 을 미디어 저장소 컬렉션에 제공 할 수 있습니다 .** 미디어 저장소는 파일 유형에 따라 **DCIM, 영화, 다운로드 등과 같이 잘 구성된 폴더에 파일을 저장**합니다. 이러한 모든 파일에 대해 앱은 File API를 통해 계속 액세스 할 수 있습니다. OS는 앱을 각 미디어 저장소 파일에 부여하는 시스템을 유지 관리하므로 앱은 저장소 권한이 없어도 원래 미디어 저장소에 기여한 파일을 읽거나 쓸 수 있습니다.

------

#### Media Store DATA column 이 deprecated된 후 사용에 대한 지침은 무엇입니까 ?

- Android 10에서 범위가 지정된 스토리지 환경의 앱은 파일 경로를 사용하여 파일에 액세스 할 수 없습니다. 이 디자인과 일관성을 유지하기 위해 DATA 열은 더 이상 사용되지 않습니다. 기존 네이티브 코드 또는 라이브러리로 작업해야 할 필요성에 대한 사용자 의견에 따라 **Android 11은 이제 범위가 지정된 저장소의 앱에 대한 파일 경로 액세스를 지원합니다. 따라서 DATA 열은 실제로 일부 시나리오에 유용 할 수 있습니다**. 미디어 저장소에 삽입하고 업데이트 하려면 범위가 지정된 저장소를 사용하는 앱은 **DISPLAY_NAME 및 RELATIVE_PATH 열을 사용**해야합니다. 더 이상이를 위해 DATA 열을 사용할 수 없습니다. 디스크에 존재하는 파일에 대한 미디어 저장소 항목을 읽을 때 DATA 열에는 파일 API 또는 NDK 파일 라이브러리와 함께 사용할 수있는 유효한 파일 경로가 있습니다. 그러나 앱은 이러한 작업으로 인한 파일 I / O 오류를 처리 할 수 있도록 준비해야하며 파일을 항상 사용할 수 있다고 가정해서는 안됩니다.

------

#### Scoped Storage를 사용하지 않는 앱의 경우 언제 Scoped Storage와 호환되어야 합니까?

- Android 11 이상을 실행하는 기기에서는 Android 11 이상을 대상으로하는 즉시 앱이 범위 저장소에 저장됩니다.

------

#### 현재 범위 스토리지 외부에 저장 한 데이터를 마이그레이션하는 권장 방법은 무엇입니까?

- **preserveLegacyExternalStorage 플래그를 사용하면 앱이 Android 11을 대상으로하는 동안에도 업그레이드시 레거시 스토리지 액세스를 유지할 수 있습니다.** 그러나 Android 11에 **새로 설치할 때는이 플래그가 적용되지 않습니다.** Android 11을 타겟팅하기 전에 스코프 스토리지에 맞게 코드를 변경하십시오. 
- 데이터 마이그레이션 모범 사례에 [대해 자세히 알아보십시오](https://developer.android.com/training/data-storage/use-cases#maintain_access_to_the_legacy_storage_location_for_data_migration) .

------

#### 앱 스토어와 같은 일부 패키지 설치 관리자가 액세스 해야하는 경우 Android / obb 디렉토리에 대한 예외가 있습니까?

- [REQUEST_INSTALL_PACKAGES ](https://developer.android.com/reference/android/Manifest.permission#REQUEST_INSTALL_PACKAGES)권한이 있는 앱은 다른 앱의 안드로이드 / OBB 디렉토리에 액세스 할 수 있습니다.