## フォーク元からの改変内容

このリポジトリは、元のLPPS ZMKドライバに下記の変更を加えたフォークです。

- 手動でキャリブレーションを要求する `&ps_calib` behavior を追加しました。
- センサ生値の調査用に `CONFIG_LPPS_SENSOR_LOG` を追加しました。有効化すると、各モーションサンプルを `LPPS_SENSOR t=<ms> x=<x> y=<y> z=<z>` の形式でINFOログに出力します。

# 低消費電力ポインティングスティック

SK8707のセンサーをI2Cバス経由で読み取れるようにしたモジュールです。元のモジュールに比べるとアクティブ電流は1/6、待機電流は1/10くらいになっています。

[ハードウェア](https://github.com/sekigon-gonnoc/low-power-pointing-stick)

## デバイスツリーの例

```dtsi
/{
    pointing_listener: pointing_listener {
        compatible = "zmk,input-listener";
        status = "okay";
    };
}

&pinctrl {
    i2c0_default: i2c0_default {
        group1 {
            psels = <NRF_PSEL(TWIM_SDA, 0, 18)>,
                    <NRF_PSEL(TWIM_SCL, 0, 16)>;
        };
    };

    i2c0_sleep: i2c0_sleep {
        group1 {
            psels = <NRF_PSEL(TWIM_SDA, 0, 18)>,
                    <NRF_PSEL(TWIM_SCL, 0, 16)>;
        };
    };
};

&i2c0 {
    status = "okay";
    compatible = "nordic,nrf-twim";
    pinctrl-0 = <&i2c0_default>;
    pinctrl-1 = <&i2c0_sleep>;
    pinctrl-names = "default", "sleep";
    clock-frequency = <I2C_BITRATE_FAST>;

    pointing_device: pointing_device@57 {
        status = "okay";
        compatible = "lpps";
        reg = <0x57>;
        irq-gpios = <&gpio0 20 GPIO_PULL_UP>;
    };
};
```

## 強制キャリブレーション

通常は自動的にキャリブレーションされますが、ドリフトが収まらない場合にはキーマップに配置した下記behaviorでキャリブレーションすることもできます。

```
#include <behaviors/lpps-calibration.dtsi>
```

`&ps_calib`
