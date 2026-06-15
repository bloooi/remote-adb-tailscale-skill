# Remote ADB Tailscale Skill

[English](README.md) | [한국어](README-ko.md)

Tailscale을 통해 Android 기기에 원격 ADB로 연결하기 위한 Codex 스킬입니다.

이 스킬은 Android Wireless Debugging으로 초기 연결을 만든 뒤, 기기의 Tailscale IP에서 TCP ADB `5555` 포트로 안정적으로 접속하도록 전체 흐름을 안내합니다.

## 무엇을 위한 스킬인가요?

`remote-adb-tailscale`은 Tailscale tailnet 안에서 접근 가능한 Android 기기에 원격 ADB로 연결할 때 사용합니다.

최종 연결 대상은 다음 형태입니다.

```text
<android-tailscale-ip>:5555
```

Wireless Debugging은 부트스트랩 단계에서만 사용합니다. 임시 Wireless Debugging 포트로 페어링하고 연결한 뒤, 스킬은 ADB를 TCP 모드 `5555` 포트로 전환하고 해당 최종 연결에서 shell 명령이 실제로 실행되는지 검증합니다.

## 요구 사항

- 개발자 옵션이 활성화된 Android 기기.
- Android 기기에서 USB debugging과 Wireless Debugging 활성화.
- 호스트 컴퓨터에 `adb` 설치.
- 호스트 컴퓨터에 Tailscale 설치 및 실행.
- Android 기기에 Tailscale 앱 설치 및 실행.
- 호스트 컴퓨터와 Android 기기가 같은 Tailscale 계정 또는 같은 tailnet에 추가되어 있어야 함.

Android 기기에는 반드시 Tailscale 앱이 설치되어 있어야 합니다. 이 방식은 ADB를 실행하는 호스트와 Android 기기가 같은 Tailscale 계정 또는 tailnet에 연결되어 있을 때만 동작합니다.

## 보안 주의사항

이 워크플로는 Tailscale tailnet을 활용합니다. 포트 `5555`가 공개 인터넷에 노출되지 않는다고 해서 안전하다고 가정하면 안 됩니다.

사용하기 전에 Tailscale Admin Console에 들어가 Access controls / ACLs 설정을 확인하세요. Android 기기가 tailnet의 모든 사용자나 모든 기기에 열려 있다면, 의도한 계정, 데스크톱 기기, 또는 신뢰할 수 있는 admin 태그가 붙은 기기만 접근할 수 있도록 ACL 규칙을 제한해야 안전합니다.

권장 사항은 다음과 같습니다.

- Android 기기를 tailnet의 모든 사용자나 모든 기기에 열어두지 마세요.
- 원격 ADB를 사용할 특정 계정 또는 특정 기기만 접근 가능하도록 Tailscale ACL에서 제한하세요.
- 포트 `5555`만 생각해서 넓게 허용하기보다 Android 노드 전체 접근을 제한하는 편이 안전합니다.
- ACL을 변경한 뒤 `adb -s <android-tailscale-ip>:5555 shell getprop ro.product.model` 명령으로 다시 연결을 확인하세요.

## 저장소 구성

```text
SKILL.md
agents/openai.yaml
README.md
README-ko.md
LICENSE
```

## 라이선스

MIT License입니다. 자세한 내용은 [LICENSE](LICENSE)를 참고하세요.
