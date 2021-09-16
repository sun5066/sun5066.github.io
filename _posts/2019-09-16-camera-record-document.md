
---
# Record Camera Document

---
> ## Base
> | Class Name | Description |
> |------------|:-----------:|
> | [BaseFragment] | [CameraFragment]의 부모 추상클래스 |

> ## Binding
> | Class Name | Description |
> |------------|:-----------:|
> | [AnimationBindingAdapter] | 애니메이션 관련 데이터바인딩 |
> | [CommonBindingAdapter] | 안드로이드에서 제공하는 기본위젯 데이터바인딩 |
> | [SegmentProgressBindingAdapter] | [SegmentedProgressView]의 클립, 퍼센트를 업데이트하기 위한 바인딩어댑터 |

> ## Activity
> | Class Name | Description |
> |------------|:-----------:|
> | [CameraActivity] | - |

> ## Fragment
> | Class Name | Description |
> |------------|:-----------:|
> | [CameraFragment] | 카메라 프리뷰, 녹화 상태를 UI로 보여줌 |
> | [SimpleDialog] | 공통으로 사용되는 팝업 다이얼로그 |

> ## State
> | Class Name | Description |
> |------------|:-----------:|
> | [RecordState] | 녹화상태를 가지고있으며 ViewModel에서 관리하고 View에게 상태를 알려줌 |

> ## Util
> | Class Name | Description |
> |------------|:-----------:|
> | [CameraConstants] | 카메라 모듈에서 사용하는 상수 모음 |
> | [FileUtil] | 녹화된 동영상 파일을 생성/삭제를 제어 |
> | [MutableListLiveData] | MutableLiveData + MutableList 커스텀 LiveData |

> ## ViewModel
> | Class Name | Description |
> |------------|:-----------:|
> | [CameraViewModel] | CameraFragment의 UI상태와 녹화관련 비즈니스 로직을 담당 |

> ## Widget
> | Class Name | Description |
> |------------|:-----------:|
> | [SegmentedProgressView] | 녹화된 클립 개수&녹화시간이 게이지로 보여지는 커스텀뷰 |

[BaseFragment]: #
[AnimationBindingAdapter]: #
[CommonBindingAdapter]: #
[SegmentProgressBindingAdapter]: #
[CameraActivity]: #
[CameraFragment]: #
[SimpleDialog]: #
[RecordState]: #
[CameraConstants]: #
[FileUtil]: #
[MutableListLiveData]: #
[CameraViewModel]: #
[SegmentedProgressView]: #