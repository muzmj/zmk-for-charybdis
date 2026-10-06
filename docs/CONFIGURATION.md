# Charybdis Mk2 (4x6) / ZMK 설정 매뉴얼

이 문서는 `withDongle_v2` 브랜치의 현재 하드웨어 구성과 주요 설정을 자세히 설명합니다.
GitHub 저장소 첫 화면의 `README.md`에는 프로젝트 개요와 최소한의 사용 안내만 두고,
세부 설정은 이 문서를 기준으로 관리합니다.

## 1. 하드웨어 구성

- Charybdis Mk2 4x6 split keyboard
- 좌측: nice!nano v2
- 우측: nice!nano v2 + PMW3610 트랙볼
- 중앙: nice!nano v2 dongle + OLED
- Dongle은 USB/BLE Central 역할
- 좌/우 keyboard half는 BLE peripheral 역할

## 2. 빌드 대상

`build.yaml`에는 현재 4개의 빌드 대상이 있습니다.

| Board | Shield | 역할 |
|---|---|---|
| nice_nano_v2 | charybdis_dongle dongle_display | USB/BLE Central 동글 + OLED |
| nice_nano_v2 | charybdis_left | 좌측 키보드 |
| nice_nano_v2 | charybdis_right | 우측 키보드 + 트랙볼 |
| nice_nano_v2 | settings_reset | 설정 초기화 |

동글 빌드에는 ZMK Studio용 `studio-rpc-usb-uart` snippet과 Studio 옵션이 적용됩니다.

## 3. 주요 파일

| 파일 | 역할 |
|---|---|
| `build.yaml` | firmware 빌드 대상 |
| `config/charybdis.conf` | 공통 Kconfig |
| `config/charybdis.keymap` | 키맵, HRM, pointing 설정 |
| `config/boards/shields/charybdis/charybdis_left.conf` | 좌측 설정 |
| `config/boards/shields/charybdis/charybdis_right.conf` | 우측/트랙볼 설정 |
| `config/boards/shields/charybdis/charybdis_left.overlay` | 좌측 하드웨어 설정 |
| `config/boards/shields/charybdis/charybdis_right.overlay` | SPI/PMW3610 설정 |
| `config/boards/shields/charybdis/charybdis.dtsi` | 공통 matrix/wakeup |
| `config/boards/shields/charybdis_dongle/charybdis_dongle.conf` | Central/OLED/BLE 설정 |
| `config/boards/shields/charybdis_dongle/charybdis_dongle.overlay` | OLED/trackball split/Layer 1 scroll |
| `config/west.yml` | ZMK 및 외부 모듈 |
| `Kconfig.defconfig` | shield 기본 Kconfig |

## 4. 공통 설정

파일: `config/charybdis.conf`

### ZMK Studio

```conf
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
CONFIG_ZMK_STUDIO_LOCK_ON_DISCONNECT=n
```

### Bluetooth

```conf
CONFIG_BT_CTLR_PHY_2M=n
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
CONFIG_ZMK_BATTERY_REPORT_INTERVAL=60
CONFIG_ZMK_KEYBOARD_NAME="CharybdisMK2"
```

### Pointing

```conf
CONFIG_ZMK_POINTING=y
```

### 키 스캔 디바운스

```conf
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=7
CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=7
```

### Deep Sleep

현재 공통 설정은 다음과 같습니다.

```conf
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
```

900000 ms = 15분입니다.

이 설정은 현재 공통 `charybdis.conf`에 있기 때문에 dongle에도 영향을 줄 수 있습니다.
dongle을 항상 깨어 있게 하려는 경우에는 Sleep 설정을 좌/우 keyboard shield 설정으로 이동하고
dongle에서는 Sleep을 비활성화하는 방식으로 분리하는 것이 적절합니다.

공통 matrix의 kscan에는 이미 `wakeup-source;`가 적용되어 있어 좌/우 키보드 입력을 wake source로 사용할 수 있습니다.

## 5. 좌측 키보드

`charybdis_left.conf`에서는 키보드 pointing 기능과 배터리 보고만 활성화합니다.

```conf
CONFIG_ZMK_POINTING=y
CONFIG_ZMK_BATTERY_REPORTING=y
```

EC11 encoder 관련 설정은 실제 하드웨어에 해당 부품이 없으므로 제거했습니다.

## 6. 우측 키보드 / 트랙볼

`charybdis_right.conf`:

```conf
CONFIG_SPI=y
CONFIG_INPUT=y
CONFIG_ZMK_MOUSE=y
CONFIG_ZMK_POINTING=y
CONFIG_ZMK_EXT_POWER=y

CONFIG_PMW3610_ALT=y
CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=1000
```

현재 PMW3610은 DoctorWangWang 드라이버가 아니라 `pixart,pmw3610-alt` 드라이버를 사용합니다.

### PMW3610 주요 설정

파일: `charybdis_right.overlay`

```dts
compatible = "pixart,pmw3610-alt";
spi-max-frequency = <2000000>;
irq-gpios = <&gpio0 6 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
cpi = <400>;
swap-xy;
invert-x;
invert-y;
```

주요 핀:
- SCK = GPIO0.8
- MOSI = GPIO0.17
- MISO = GPIO0.17
- CS = GPIO0.20
- IRQ = GPIO0.6

현재 CPI는 400입니다.

트랙볼 데이터는 우측 half에서 읽은 후 `trackball_split`을 통해 BLE Split 구조로 Central에 전달됩니다.

## 7. 트랙볼 커서

파일: `config/charybdis.keymap`

```c
#define ZMK_POINTING_DEFAULT_MOVE_VAL 1000
```

현재 기본 이동값은 1000입니다.

```dts
&mmv_input_listener {
    input-processors = <&zip_xy_scaler 2 1>;
};

&mmv {
    time-to-max-speed-ms = <500>;
    acceleration-exponent = <1>;
    trigger-period-ms = <16>;
};
```

의미:
- XY scaler = 2배
- 최대속도 도달시간 = 500 ms
- acceleration exponent = 1
- trigger period = 16 ms

커서가 너무 빠르거나 느리면 우선 `zip_xy_scaler`와 `ZMK_POINTING_DEFAULT_MOVE_VAL`을 조정합니다.

## 8. 일반 스크롤

```c
#define ZMK_POINTING_DEFAULT_SCRL_VAL 12
```

현재 기본 스크롤 값은 12입니다.

```dts
&msc_input_listener {
    input-processors = <&zip_scroll_scaler 2 1>;
};

&msc {
    acceleration-exponent = <0>;
    time-to-max-speed-ms = <0>;
    delay-ms = <0>;
};
```

일반 스크롤에는 2배 scaler가 적용되며 가속/지연은 사용하지 않습니다.

## 9. Layer 1 트랙볼 스크롤

동글의 `charybdis_dongle.overlay`에서 Layer 1 전용 listener를 사용합니다.

처리 순서:
```text
트랙볼 X/Y
→ Y 반전
→ XY → Scroll 변환
→ 1/40 scaler
→ Scroll HID
```

핵심 설정:
```dts
layers = <1>;
input-processors =
    <&zip_xy_transform INPUT_TRANSFORM_Y_INVERT>,
    <&zip_xy_to_scroll_mapper>,
    <&my_scroll_scaler 1 40>;
```

`my_scroll_scaler`에는 `track-remainders`가 적용되어 작은 입력의 나머지를 누적합니다.

Layer 1 스크롤 속도는 우선 `<&my_scroll_scaler 1 40>` 값을 조정합니다.

## 10. 동글

파일: `charybdis_dongle.conf`

주요 기능:
- BLE Central
- 좌/우 peripheral 2개 연결
- OLED
- peripheral 배터리 표시
- 동글 자체 배터리 표시
- WPM
- Pointing

주요 설정:
```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_IDLE_TIMEOUT=60000
CONFIG_ZMK_DONGLE_DISPLAY_LAYER=y

CONFIG_BT_MAX_CONN=6
CONFIG_BT_MAX_PAIRED=6
CONFIG_ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=2

CONFIG_BT_BAS=n
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y

CONFIG_ZMK_EXT_POWER=y
CONFIG_ZMK_BLE_PASSKEY_ENTRY=n
CONFIG_ZMK_BLE_CLEAR_BONDS_ON_START=n
CONFIG_ZMK_SETTINGS_SAVE_DEBOUNCE=10000
CONFIG_ZMK_WPM=y
CONFIG_LV_Z_MEM_POOL_SIZE=8192
CONFIG_ZMK_POINTING=y
CONFIG_ZMK_DONGLE_DISPLAY_DONGLE_BATTERY=y
```

동글에는 encoder와 RGB 관련 설정이 없습니다.

## 11. HRM

현재 `config/charybdis.keymap`의 global hold-tap 설정:

```dts
&lt {
    tapping-term-ms = <200>;
    flavor = "balanced";
    quick-tap-ms = <150>;
};
```

현재 HRM:
- A = Left Shift
- S = Left Ctrl
- D = Left Alt
- F = Left GUI
- J = Right GUI
- K = Left Alt
- L = Left Ctrl
- ; = Right Shift

중요: 현재 소스의 flavor는 `balanced`입니다. 이전 설정에서 사용했던 `hold-preferred`와 다릅니다.

HRM 오동작이 발생하면 우선 tapping term, flavor, quick-tap 값을 확인합니다.

## 12. 외부 모듈

`config/west.yml`의 핵심 모듈:

| 모듈 | 버전/브랜치 | 용도 |
|---|---|---|
| ZMK | v0.3 | 키보드 firmware |
| zmk-pmw3610-driver | main | PMW3610 트랙볼 |
| zmk-dongle-display | v0.3 | 동글 OLED |

## 13. 펌웨어 업데이트 / 복구 시 확인 항목

upstream 변경 후 동작이 달라질 경우 다음 항목을 우선 비교합니다.

1. `build.yaml`의 4개 build target
2. `charybdis_right.conf`의 PMW3610-alt 설정
3. `charybdis_right.overlay`의 SPI/PMW3610 설정
4. CPI 400 및 `swap-xy`, `invert-x`, `invert-y`
5. `trackball_split`
6. Layer 1 `my_scroll_scaler 1 40`
7. `DEFAULT_MOVE_VAL=1000`
8. `zip_xy_scaler 2 1`
9. `DEFAULT_SCRL_VAL=12`
10. `zip_scroll_scaler 2 1`
11. `zmk-pmw3610-driver`
12. `wakeup-source`
13. Sleep 설정

특히 PMW3610 관련 설정은 기존 DoctorWangWang 방식과 다르므로 upstream 동기화 후 반드시 확인합니다.

## 14. 참고

- ZMK Pointing: https://zmk.dev/docs/hardware-integration/pointing
- ZMK Input Processors: https://zmk.dev/docs/keymaps/input-processors
- ZMK Scaler: https://zmk.dev/docs/keymaps/input-processors/scaler
- ZMK Mouse/Pointing Behavior: https://zmk.dev/docs/keymaps/behaviors/mouse-emulation
