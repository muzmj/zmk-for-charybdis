# Charybdis Mk2 (4x6) / ZMK

## 1. 빌드 구성

`build.yaml`에는 4개의 빌드 대상이 있습니다.

---

대상 보드 Shield 역할

---

1 `nice_nano_v2` `charybdis_dongle dongle_display` USB/BLE Central 동글 + OLED

2 `nice_nano_v2` `charybdis_left` 좌측 키보드

3 `nice_nano_v2` `charybdis_right` 우측 키보드 + 트랙볼

4 `nice_nano_v2` `settings_reset` 설정 초기화

---

동글 빌드에는 `studio-rpc-usb-uart`와 ZMK Studio 옵션이 적용됩니다.

---

## 2. 공통 `.conf` --- `config/charybdis.conf`

### ZMK Studio

```conf
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
CONFIG_ZMK_STUDIO_LOCK_ON_DISCONNECT=n
```

### Bluetooth / 무선

```conf
CONFIG_BT_CTLR_PHY_2M=n
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y
CONFIG_ZMK_BATTERY_REPORT_INTERVAL=60
CONFIG_ZMK_KEYBOARD_NAME="CharybdisMK2"
```

현재 주석 처리된 Bluetooth 튜닝값도 있습니다.

```conf
# CONFIG_BT_LL_SW_LLCP_LEGACY=y
# CONFIG_BT_PERIPHERAL_PREF_MAX_INT=9
# CONFIG_BT_PERIPHERAL_PREF_LATENCY=16
# CONFIG_BT_BUF_ACL_TX_COUNT=32
# CONFIG_BT_L2CAP_TX_BUF_COUNT=32
```

### RGB

```conf
CONFIG_ZMK_RGB_UNDERGLOW=y
CONFIG_WS2812_STRIP=y
CONFIG_LED_STRIP=y
CONFIG_ZMK_RGB_UNDERGLOW_ON_START=y
CONFIG_ZMK_RGB_UNDERGLOW_EFF_START=3
```

- 좌측 RGB: 29 LEDs
- 우측 RGB: 27 LEDs
- 동글은 실제 RGB가 없어 동글 `.conf`에서 비활성화

### Encoder / Pointing

```conf
CONFIG_EC11=y
CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y
CONFIG_ZMK_POINTING=y
```

### 키 스캔 디바운스

```conf
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=7
CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=7
```

### Deep Sleep

```conf
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
```

`900000 ms = 15분`입니다.

**중요:** 이 설정은 공통 `charybdis.conf`에 있으므로 동글에도 공통
Kconfig로 전달될 수 있습니다. 동글 `.conf`의 Sleep 설정은 주석 처리되어
있지만, 공통 설정을 자동으로 무효화하는 것은 아닙니다. 동글을 항상 깨어
있게 하려면 이 부분을 별도로 확인할 필요가 있습니다.

---

## 3. 좌측 키보드 --- `charybdis_left.conf`

현재 설정:

```conf
CONFIG_EC11=y
CONFIG_EC11_TRIGGER_GLOBAL_THREAD=y
CONFIG_ZMK_POINTING=y
```

`charybdis_left.overlay`에서 좌측 encoder를 활성화합니다.

```dts
&left_encoder_1 {
    status = "okay";
};
```

encoder GPIO:

```text
A = GPIO1.11
B = GPIO1.13
steps = 48
triggers-per-rotation = 24
```

좌측 RGB는:

```dts
chain-length = <29>;
```

입니다.

---

## 4. 우측 키보드 --- `charybdis_right.conf`

```conf
CONFIG_SPI=y
CONFIG_INPUT=y
CONFIG_ZMK_MOUSE=y
CONFIG_ZMK_POINTING=y
CONFIG_ZMK_EXT_POWER=y

CONFIG_PMW3610_ALT=y
CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=1000
```

현재 트랙볼은 **DoctorWangWang 드라이버가 아니라 `pmw3610-alt`
드라이버**를 사용합니다.

PMW3610 초기화 시 추가 전원 대기시간은 1000 ms입니다.

---

# 5. 트랙볼 하드웨어 설정

파일:

`config/boards/shields/charybdis/charybdis_right.overlay`

## SPI

```dts
&spi0 {
    status = "okay";
    compatible = "nordic,nrf-spim";
    pinctrl-0 = <&spi0_default>;
    pinctrl-1 = <&spi0_sleep>;
    pinctrl-names = "default", "sleep";
    cs-gpios = <&gpio0 20 GPIO_ACTIVE_LOW>;
```

SPI 관련 핀:

```text
SCK  = GPIO0.8
MOSI = GPIO0.17
MISO = GPIO0.17
CS   = GPIO0.20
```

## PMW3610

```dts
trackball: trackball@0 {
    status = "okay";
    compatible = "pixart,pmw3610-alt";
    reg = <0>;
    spi-max-frequency = <2000000>;
    irq-gpios = <&gpio0 6 (GPIO_ACTIVE_LOW | GPIO_PULL_UP)>;
    cpi = <400>;
    swap-xy;
    invert-x;
    invert-y;
    evt-type = <INPUT_EV_REL>;
    x-input-code = <INPUT_REL_X>;
    y-input-code = <INPUT_REL_Y>;
};
```

### 주요 값

항목 값

---

드라이버 `pixart,pmw3610-alt`
SPI 최대 속도 2 MHz
IRQ GPIO0.6
CPI **400**
X/Y Swap 활성
X 반전 활성
Y 반전 활성
출력 Relative X/Y

현재 소스에는 `pixart` vendor-prefix 관련 빌드 경고가 나타날 수 있지만,
이전 빌드 로그에서는 이것이 실패 원인이 아니라 warning이었습니다.

---

# 6. 트랙볼 Split 구조

우측에서:

```dts
trackball_split: trackball_split@0 {
    compatible = "zmk,input-split";
    reg = <0>;
    device = <&trackball>;
};
```

구조:

```text
PMW3610
  ↓
SPI0
  ↓
pmw3610-alt
  ↓
trackball
  ↓
trackball_split
  ↓
BLE Split
  ↓
Dongle / Central
  ↓
PC
```

즉 트랙볼은 우측 nice!nano에서 읽고 동글 Central로 전달됩니다.

---

# 7. 트랙볼 커서 속도

파일:

`config/charybdis.keymap`

### 기본 이동값

```c
#define ZMK_POINTING_DEFAULT_MOVE_VAL 1000
```

ZMK 기본값 600 대신 1000을 사용합니다.

### XY scaler

```dts
&mmv_input_listener {
    input-processors = <&zip_xy_scaler 2 1>;
};
```

`zip_xy_scaler 2 1`은 입력값을 `2/1`, 즉 **2배**로 만듭니다.

따라서 현재 커서 이동은:

```text
PMW3610 CPI 400
→ 기본 pointing move 값 1000
→ XY scaler 2배
→ mouse movement
```

구조입니다.

---

# 8. 커서 가속

```dts
&mmv {
    time-to-max-speed-ms = <500>;
    acceleration-exponent = <1>;
    trigger-period-ms = <16>;
};
```

설정 값 의미

---

최대속도 도달시간 500 ms 0.5초 동안 가속
acceleration exponent 1 일반적인 선형 가속
trigger period 16 ms 약 60 Hz

따라서 단순히 `MOVE_VAL`만으로 커서 속도가 결정되는 구조가 아닙니다.

---

# 9. 트랙볼 일반 스크롤 설정

### 기본 스크롤 값

```c
#define ZMK_POINTING_DEFAULT_SCRL_VAL 12
```

기본값 10 대신 12입니다.

### 일반 scroll scaler

```dts
&msc_input_listener {
    input-processors = <&zip_scroll_scaler 2 1>;
};
```

스크롤 입력에도 **2배** scaler가 적용됩니다.

### 스크롤 가속

```dts
&msc {
    acceleration-exponent = <0>;
    time-to-max-speed-ms = <0>;
    delay-ms = <0>;
};
```

즉:

- 가속 없음
- 최대속도 도달시간 0
- 입력 지연 0

으로 즉각적인 스크롤을 사용합니다.

---

# 10. Layer 1 트랙볼 스크롤

동글의 `charybdis_dongle.overlay`에 별도의 트랙볼 listener가 있습니다.

```dts
trackball_listener: trackball_listener {
    compatible = "zmk,input-listener";
    device = <&trackball_split>;

    scroller {
        layers = <1>;
        input-processors =
            <&zip_xy_transform INPUT_TRANSFORM_Y_INVERT>,
            <&zip_xy_to_scroll_mapper>,
            <&my_scroll_scaler 1 40>;
    };
};
```

Layer 1에서는 다음 순서로 처리됩니다.

```text
트랙볼 X/Y
→ Y 반전
→ XY를 Scroll로 변환
→ 1/40 scaler
→ Scroll HID
```

`my_scroll_scaler`:

```dts
my_scroll_scaler: my_scroll_scaler {
    compatible = "zmk,input-processor-scaler";
    #input-processor-cells = <2>;
    type = <INPUT_EV_REL>;
    codes = <INPUT_REL_WHEEL INPUT_REL_HWHEEL>;
    track-remainders;
};
```

실제 값:

```dts
<&my_scroll_scaler 1 40>
```

즉 Layer 1 물리 트랙볼 스크롤은 **1/40 배율**입니다.

`track-remainders` 때문에 나눗셈으로 바로 버려지는 작은 입력의 나머지를
추적하여 누적할 수 있습니다.

---

# 11. 트랙볼 속도 설정 요약

항목 현재값

---

PMW3610 CPI **400**
`DEFAULT_MOVE_VAL` **1000**
커서 XY scaler **2/1**
커서 가속 **1**
커서 최대속도 도달 **500 ms**
커서 trigger period **16 ms**
`DEFAULT_SCRL_VAL` **12**
일반 scroll scaler **2/1**
일반 scroll acceleration **0**
일반 scroll max-speed time **0 ms**
Layer 1 Y invert 적용
Layer 1 XY→Scroll 적용
Layer 1 scroll scaler **1/40**

### 속도를 조절할 때 우선 볼 값

커서가 너무 빠르면:

```dts
<&zip_xy_scaler 2 1>
```

또는:

```c
#define ZMK_POINTING_DEFAULT_MOVE_VAL 1000
```

을 확인합니다.

Layer 1 스크롤이 너무 빠르거나 느리면:

```dts
<&my_scroll_scaler 1 40>
```

을 먼저 조절하는 것이 직접적입니다.

---

# 12. 동글 설정

파일:

`config/boards/shields/charybdis_dongle/charybdis_dongle.conf`

### RGB / Encoder

```conf
CONFIG_ZMK_RGB_UNDERGLOW=n
CONFIG_WS2812_STRIP=n
CONFIG_LED_STRIP=n
CONFIG_EC11=n
```

동글에는 RGB와 encoder를 사용하지 않습니다.

### OLED

```conf
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_IDLE_TIMEOUT=60000
CONFIG_ZMK_DONGLE_DISPLAY_LAYER=y
```

OLED idle timeout은 60초입니다.

### Bluetooth / Split Central

```conf
CONFIG_BT_MAX_CONN=6
CONFIG_BT_MAX_PAIRED=6
CONFIG_ZMK_SPLIT_BLE_CENTRAL_PERIPHERALS=2
```

좌/우 2개 peripheral을 연결하는 Central입니다.

추가로:

```conf
CONFIG_BT_BAS=n
CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_FETCHING=y
```

으로 peripheral 배터리 정보를 가져옵니다.

### 기타

```conf
CONFIG_ZMK_EXT_POWER=y
CONFIG_ZMK_BLE_PASSKEY_ENTRY=n
CONFIG_ZMK_BLE_CLEAR_BONDS_ON_START=n
CONFIG_ZMK_SETTINGS_SAVE_DEBOUNCE=10000
CONFIG_ZMK_WPM=y
CONFIG_LV_Z_MEM_POOL_SIZE=8192
CONFIG_ZMK_POINTING=y
CONFIG_ZMK_DONGLE_DISPLAY_DONGLE_BATTERY=y
```

---

# 13. Wake / Deep Sleep

공통 `charybdis.dtsi`의 matrix kscan에는 이미:

```dts
kscan0: kscan {
    compatible = "zmk,kscan-gpio-matrix";

    wakeup-source;
    ...
};
```

가 있습니다.

따라서 좌/우 키보드의 matrix 입력을 wake source로 사용할 수 있도록 되어
있습니다.

현재 Sleep:

```conf
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000
```

= 15분입니다.

좌/우 MCU는 서로 독립적이므로, 한쪽 키보드의 키 입력으로 다른 쪽 MCU까지
직접 깨우려면 별도의 하드웨어 wake 경로가 필요합니다. 현재 구조에서는 각
half가 자신의 matrix 입력으로 wake하는 방식이 기본입니다.

---

# 14. 현재 HRM 설정

`config/charybdis.keymap`:

```dts
&lt {
    tapping-term-ms = <200>;
    flavor = "balanced";
    quick-tap-ms = <150>;
};
```

현재:

항목 값

---

Tapping term 200 ms
Flavor `balanced`
Quick tap 150 ms

Home Row Mods:

```text
A = Left Shift
S = Left Ctrl
D = Left Alt
F = Left GUI

J = Right GUI
K = Left Alt
L = Left Ctrl
; = Right Shift
```

현재 소스의 `flavor`는 **`balanced`**입니다. 이전에 사용했던
`hold-preferred`와 다르므로 HRM 오동작 조정 시 반드시 현재 소스를
기준으로 확인해야 합니다.

---

# 15. Encoder 설정

세로 스크롤:

```dts
bindings = <&msc SCRL_DOWN>, <&msc SCRL_UP>;
tap-ms = <100>;
```

가로 스크롤:

```dts
bindings = <&msc SCRL_LEFT>, <&msc SCRL_RIGHT>;
tap-ms = <100>;
```

현재 encoder는 스크롤 용도로 사용됩니다.

---

# 16. 외부 모듈

`config/west.yml`:

```yaml
projects:
  - name: zmk
    remote: zmkfirmware
    revision: v0.3
    import: app/west.yml

  - name: zmk-pmw3610-driver
    remote: badjeff
    revision: main

  - name: zmk-dongle-display
    remote: englmaxi
    revision: v0.3
```

핵심 의존성:

모듈 버전/브랜치 용도

---

ZMK `v0.3` 키보드 펌웨어
`zmk-pmw3610-driver` `main` PMW3610 트랙볼
`zmk-dongle-display` `v0.3` 동글 OLED

---

# 17. 파일별 빠른 참조

파일 역할

- `build.yaml` 4개 firmware 빌드
- `config/charybdis.conf` 공통 Kconfig
- `config/charybdis.keymap` 키맵, HRM, encoder, pointing 속도
- `charybdis_left.conf` 좌측 encoder/pointing
- `charybdis_right.conf` PMW3610 driver/SPI/pointing
- `charybdis_left.overlay` 좌측 matrix/encoder/RGB
- `charybdis_right.overlay` PMW3610/SPI/RGB/input split
- `charybdis.dtsi` 공통 matrix/wakeup/encoder
- `charybdis_dongle.conf` Central/OLED/BLE/display
- `charybdis_dongle.overlay` OLED/trackball split/Layer 1 scroll
- `config/west.yml` ZMK/외부 모듈
- `Kconfig.defconfig` shield 기본 Kconfig

---

# 18. 향후 복구 시 가장 중요한 항목

upstream 업데이트로 설정이 덮어써질 경우 다음 순서로 비교하면 현재
동작을 복원하기 쉽습니다.

1.  `charybdis_right.conf`의 `CONFIG_PMW3610_ALT=y`
2.  `charybdis_right.conf`의
    `CONFIG_PMW3610_ALT_INIT_POWER_UP_EXTRA_DELAY_MS=1000`
3.  `charybdis_right.overlay`의 PMW3610 설정
4.  `cpi = <400>`
5.  `swap-xy`, `invert-x`, `invert-y`
6.  `charybdis_dongle.overlay`의 `trackball_split`
7.  Layer 1의 `my_scroll_scaler 1 40`
8.  `charybdis.keymap`의 `DEFAULT_MOVE_VAL=1000`
9.  `zip_xy_scaler 2 1`
10. `DEFAULT_SCRL_VAL=12`
11. `zip_scroll_scaler 2 1`
12. `west.yml`의 `zmk-pmw3610-driver`
13. `build.yaml`의 4개 target
14. `charybdis.dtsi`의 `wakeup-source`
15. Sleep 설정

---

## 참고 문서

- Repository:
  https://github.com/muzmj/zmk-for-charybdis/tree/withDongle_v2
- ZMK Pointing Devices:
  https://zmk.dev/docs/hardware-integration/pointing
- ZMK Input Processors: https://zmk.dev/docs/keymaps/input-processors
- ZMK Scaler: https://zmk.dev/docs/keymaps/input-processors/scaler
- ZMK Mouse/Pointing Behavior:
  https://zmk.dev/docs/keymaps/behaviors/mouse-emulation
