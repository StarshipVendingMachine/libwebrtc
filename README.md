# libwebrtc [![License][license-img]][license-href]  

## `윈도우 C++/CLI빌드용 Webrtc Dynamic Build`
- 기준버전: `WEBRTC - Release 60버전`
- 직접빌드 이유: `윈도우버전은 static build 및 vp8, vp9 codec의 사용이 기본이며, h264 미포함
is_component_build 값에 따라 DLL이 생성되며, assert값으로 빌드가 막혀 있으므로 주석처리해야함`
- 수동빌드 참조사이트: http://webrtc.github.io/webrtc-org/native-code/development/

## `해당 프로젝트의 빌드방법`
```
1) git clone https://github.com/StarshipVendingMachine/libwebrtc.git
2) cd libwebrtc
3) md out
4) cd out
5) cmake -DCMAKE_BUILD_TYPE=Debug -DTARGET_CPU=x86 -A Win32 ..
```

## `수동 빌드 빌드옵션(OPEN H264추가) - 테스트완료`

- `Debug Build Option:`
  - gn gen out/win_x86_debug -args="is_debug=true target_cpu=\"x86\" is_component_build=true proprietary_codecs=true rtc_use_h264=true use_openh264=true ffmpeg_branding=\"Chrome\" rtc_include_tests=false rtc_include_pulse_audio=false use_sysroot=false is_clang=false treat_warnings_as_errors=false" --ide=vs
  - ninja -C out/win_x86_debug


- `Release Build Option:`
  - gn gen out/win_x86_release -args="is_debug=false symbol_level=0 enable_nacl=false target_cpu=\"x86\" is_component_build=true proprietary_codecs=true rtc_use_h264=true use_openh264=true ffmpeg_branding=\"Chrome\" rtc_include_tests=false rtc_include_pulse_audio=false use_sysroot=false is_clang=false treat_warnings_as_errors=false" --ide=vs
  - ninja -C out/win_x86_release

## `빌드완료 후 필수파일`
```
boringssl.dll
ffmpeg.dll
protobuf_lite.dll
turbojpeg.dll - `External dll`
```

## `릴리즈빌드에서 오류발생`
- Access Violation 발생시 수정
- Path: `.\gen\webrtc\logging\rtc_event_log\rtc_event_log.pb.h`

```c++
inline void DecoderConfig::set_name(const ::std::string& value) {
  set_has_name();
  std::string copiedValue = value; // <- 추가 (수정항목)
  name_.SetNoArena(&::google::protobuf::internal::GetEmptyStringAlreadyInited(), copiedValue);
  // @@protoc_insertion_point(field_set:webrtc.rtclog.DecoderConfig.name)
}
```


---

## `M77 수동빌드 시 수정 및 에러처리`

1. Win32 오류발생 - 모듈없어서 발생
    - 'pip install pypiwin32'
2. 아래의 코드로 인해 h264빌드에 오류발생 가능
    - rtc_use_h264 = proprietary_codecs && !is_android && !is_ios && !(is_win && !is_clang)
3. `See: bugs.webrtc.org/9213#c13.` - 오류발생 시 수정할 부분 (주석처리)
    - 대상파일:
        * .\src\modules\video_coding\codecs\h264\h264_decoder_impl.h
        * .\src\modules\video_coding\codecs\h264\h264_encoder_impl.h
        * .\src\modules\video_coding\codecs\h264\h264_color_space.h

    - PCM_VIDC FLAG 없어서 발생하는 오류
        * .\src\third_party\ffmpeg\libavcodec\pcm.c
