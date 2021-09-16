
---
# Video Trim Document

---

> ## Base
> | Class Name | Description |
> | ---------- | ----------- |
> | [BaseFragment] | - |
> | [BaseViewModel] | - |
---
> ## DataBinding
> | Class Name | Description |
> | ---------- | ----------- |
> | [CommonBindingAdapter] | 안드로이드에서 제공하는 기본위젯 데이터바인딩 |
> | [LottieBindingAdapter] | 로티 애니메이션 데이터바인딩 |
> | [RangeViewBindingAdapter] | RangeView 데이터바인딩 |
---
> ## Fragment
> | Class Name | Description |
> | ---------- | ----------- |
> | [CancelDialog] | 동영상 편집, 업로드 중일때 물리 백버튼 클릭시 띄우는 팝업 |
> | [VideoTrimFragment] | 동영상 구간편집을 화면으로 담당 |
---
> ## DTO Model
> | Class Name | Description |
> | ---------- | ----------- |
> | [MediaInfo] | DB에서 가져온 미디어정보를 가지고있음 |
> | [UploadFileData] | MediaInfo 정보를 가지고 업로드할 임시파일을 가지고있음 |
> | [UploadRequestData] | UploadFileData에서 가지고 있는 임시파일을 multiPart로 가지고 있음 |
> | [UploadResultData] | 업로드 결과를 받아서 브릿지로 쏘기위한 클래스 |
---
> ## Repository Interface
> | Class Name | Description |
> | ---------- | ----------- |
> | [RestApiService] | RestApi 서비스 모음 |
---
> ## Repository Object Class
> | Class Name | Description |
> | ---------- | ----------- |
> | [RestApiRequest] | 파일 업로드 API Service와 업로드 파일을 multiPart로 변경을 담당 |
---
> ## State
> | Class Name | Description |
> | ---------- | ----------- |
> | [PlayerState] | 프리뷰 플레이어의 재생/일시정지 상태를 가지고있음 |
> | [VideoTrimState] | VideoTrimFragment가 현재 영상 편집, 업로드 상태를 가지고있음 |
---
> ## UseCase
> | Class Name | Description |
> | ---------- | ----------- |
> | [FileUploadUseCase] | 업로드 service를 생성자로 받아서 "업로드만" 처리해주는 역할 |
> | [VideoConvertUseCase] | 동영상의 해상도를 1920*1080, 1080*1920으로 변경해주는 역할 |
---
> ## Util
> | Class Name | Description |
> | ---------- | ----------- |
> | [RestConstants].[ImageConfig] | 이미지 업로드 API 관련 상수 모음 |
> | [RestConstants].[VideoConfig] | 동영상 업로드 API 관련 상수 모음 |
> | [ModuleCommonConstants] | 프로젝트의 모든 모듈에서 사용할 상수 모음 |
> | [VideoTrimmerConstants] | 동영상 구간편집에서 사용할 상수 모음 |
> | [FileUtil] | 파일관련 비즈니스 로직 모음 |
> | [MediaFileUploader] | 비디오 서버에 업로드를 담당(업로드전 동영상 해상도 변경, 사진 Resize도 여기서 해주고있음) |
> | [MutableListLiveData] | MutableList를 LiveData로 관리 |
> | [Resizer] | 업로드할 사진 크기, 퀄리티를 담당 |
> | [VideoCutter] | 영상 구간편집을 담당 |
---
> ## ViewModel
> | Class Name | Description |
> | ---------- | ----------- |
> | [SharedViewModel] | 각 화면마다 backPressListener를 Stack으로 관리해줌 |
> | [VideoTrimViewModel] | [VideoTrimFragment]의 UI 상태와 영상 편집/업로드 관련 비즈니스 로직을 담당 |
---
> ## Widget
> | Class Name | Description |
> | ---------- | ----------- |
> | [DraggingState] | [RangeView]를 터치 상태를 가지고있음 |
> | [RangeView] | 영상 구간편집을 하기위한 UI 위젯 |

[BaseFragment]: #
[BaseViewModel]: #
[CommonBindingAdapter]: #
[LottieBindingAdapter]: #
[RangeViewBindingAdapter]: #
[CancelDialog]: #
[VideoTrimFragment]: #
[MediaInfo]: #
[UploadFileData]: #
[UploadRequestData]: #
[UploadResultData]: #
[RestApiService]: #
[RestApiRequest]: #
[PlayerState]: #
[VideoTrimState]: #
[FileUploadUseCase]: #
[VideoConvertUseCase]: #
[RestConstants]: #
[ImageConfig]: #
[RestConstants]: #
[VideoConfig]: #
[ModuleCommonConstants]: #
[VideoTrimmerConstants]: #
[FileUtil]: #
[MediaFileUploader]: #
[MutableListLiveData]: #
[Resizer]: #
[VideoCutter]: #
[SharedViewModel]: #
[VideoTrimViewModel]: #
[DraggingState]: #
[RangeView]: #