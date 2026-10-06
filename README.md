# Charybdis Mk2 (4x6) / ZMK

Charybdis Mk2 4x6 split keyboard용 ZMK firmware 설정입니다.

현재 `withDongle_v2` 구성은 **좌/우 키보드 + BLE Central 동글** 구조로 동작하며, 우측 키보드에는 PMW3610 트랙볼이 포함되어 있습니다.

## 주요 구성

- **Left** — Charybdis 좌측 키보드
- **Right** — Charybdis 우측 키보드 + PMW3610 트랙볼
- **Dongle** — BLE Central + OLED
- **Settings Reset** — ZMK 설정 초기화용
- ZMK Studio 지원
- Layer 1 트랙볼 스크롤
- Home Row Mods 지원
- PMW3610용 `zmk-pmw3610-driver` 사용

현재 `build.yaml`에는 위 구성에 필요한 **4개 firmware 대상**이 포함되어 있습니다.

## 상세 설정 매뉴얼

하드웨어 구성, 빌드 대상, PMW3610 트랙볼 설정, 커서/스크롤 속도, Layer 1 스크롤, HRM, Deep Sleep, OLED/BLE 동글 설정 및 upstream 업데이트 후 복구 시 확인할 항목은 아래 문서에 정리되어 있습니다.

**[설정 및 유지보수 매뉴얼](docs/CONFIGURATION.md)**

## 기본 빌드 구성

`build.yaml`:

| 대상 | 역할 |
|---|---|
| `charybdis_dongle dongle_display` | BLE Central + OLED |
| `charybdis_left` | 좌측 키보드 |
| `charybdis_right` | 우측 키보드 + 트랙볼 |
| `settings_reset` | 설정 초기화 |

자세한 빌드/플래싱 및 설정 변경 방법은 **[설정 및 유지보수 매뉴얼](docs/CONFIGURATION.md)**을 참고하세요.

## 참고

- [ZMK Pointing Devices](https://zmk.dev/docs/hardware-integration/pointing)
- [ZMK Input Processors](https://zmk.dev/docs/keymaps/input-processors)
- [ZMK Scaler](https://zmk.dev/docs/keymaps/input-processors/scaler)
- [ZMK Mouse/Pointing Behavior](https://zmk.dev/docs/keymaps/behaviors/mouse-emulation)
