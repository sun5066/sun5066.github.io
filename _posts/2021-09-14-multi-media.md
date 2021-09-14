---
title: "안드로이드에서 영상 녹화&편집까지 해보자"
date: 2021-09-14 09:43:00 -0400
categories: android-yeoboya
---

---
## 사용된 공통 라이브러리
- AAC
- Coroutine 1.3

### VideoTrim (구간편집/업로드 모듈)
- ExoPlayer2
- Retrofit 2
- OkHttp 3
- FFMPEG

### Camera Record (영상 녹화 모듈)
- CameraRecorder-android v0.1.5
- mp4parser-muxer 1.9
---

# 설명하기 앞서서 말씀드리자면..
우선 해당 라이브러리는 모든 Activity/Fragment는 **com.lib.multiimagechooser.MediaDialogFragment.kt** 에서 LifeCycle, BackPress를 제어하고 있기 때문에 잘 기억해놓으셔야합니다.

---
- # Video Trim Document
---
> ## Base
> | Class Name | Description |
> | ---------- | ----------- |
> | BaseFragment | - |
> | BaseViewModel | - |

> ## DataBinding
> | Class Name | Description |
> | ---------- | ----------- |
> | CommonBindingAdapter | - |
> | LottieBindingAdapter | - |
> | RangeViewBindingAdapter | - |

> ## Fragment
> | Class Name | Description |
> | ---------- | ----------- |
> | CancelDialog | - |
> | VideoTrimFragment | - |

> ## Model
> | Class Name | Description |
> | ---------- | ----------- |
> | MediaInfo | - |
> | UploadFileData | - |
> | UploadRequestData | - |
> | UploadResultData | - |

> ## Repository Interface
> | Class Name | Description |
> | ---------- | ----------- |
> | RestApiService | - |

> ## Repository Object Class
> | Class Name | Description |
> | ---------- | ----------- |
> | RestApiRequest | - |

> ## State
> | Class Name | Description |
> | ---------- | ----------- |
> | PlayerState | - |
> | VideoTrimState | - |

> ## UseCase
> | Class Name | Description |
> | ---------- | ----------- |
> | FileUploadUseCase | - |
> | VideoConvertUseCase | - |

> ## Util
> | Class Name | Description |
> | ---------- | ----------- |
> | VideoTrimmerConstants | - |
> | RestConstants.ImageConfig | - |
> | RestConstants.VideoConfig | - |
> | ModuleCommonConstants | - |
> | FileUtil | - |
> | MediaFileUploader | - |
> | MutableListLiveData | - |
> | Resizer | - |
> | VideoCutter | - |

> ## ViewModel
> | Class Name | Description |
> | ---------- | ----------- |
> | SharedViewModel | - |
> | VideoTrimViewModel | - |

> ## Widget
> | Class Name | Description |
> | ---------- | ----------- |
> | DraggingState | - |
> | RangeView | - |
---
* # Record Camera Document
---
> ## Base
> | Class Name | Description |
> | ---------- | ----------- |
> | BaseFragment | - |

> ## Binding
> | Class Name | Description |
> | ---------- | ----------- |
> | AnimationBindingAdapter | - |
> | CommonBindingAdapter | - |
> | SegmentProgressBindingAdapter | - |

> ## Activity
> | Class Name | Description |
> | ---------- | ----------- |
> | CameraActivity | - |

> ## Fragment
> | Class Name | Description |
> | ---------- | ----------- |
> | CameraFragment | - |
> | SimpleDialog | 공통으로 사용되는 팝업 다이얼로그 |

> ## State
> | Class Name | Description |
> | ---------- | ----------- |
> | RecordState | 녹화상태를 가지고있으며 ViewModel에서 관리하고 View에게 상태를 알려줌 |

> ## Util
> | Class Name | Description |
> | ---------- | ----------- |
> | CameraConstants | 카메라 모듈에서 사용하는 상수 모음 |
> | FileUtil | 녹화된 동영상 파일을 생성/삭제를 제어 |
> | MutableListLiveData | MutableLiveData + MutableList 커스텀 LiveData |

> ## ViewModel
> | Class Name | Description |
> | ---------- | ----------- |
> | CameraViewModel | - |

> ## Widget
> | Class Name | Description |
> | ---------- | ----------- |
> | [SegmentedProgressView] | 녹화된 클립 개수&녹화시간 만큼 퍼센트가 차는 커스텀뷰 |

[SegmentedProgressView]: https://github.com/bravobit/FFmpeg-Android