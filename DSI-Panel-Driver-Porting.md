This is a work-in-progress document describing how to decipher panel driver programming from downstream android dts files, in particular 3.10 based kernels.  I'm working off of the kernel for sony xperia devices, but (I think) other 3.10 based devices should be similar.

Rather than having per-panel drivers that are decipherable, the downstream kernel uses a "meta" panel driver, with programming sequences taken from device tree (dts) files.  For example, the xperia z3 panel description for the auo novatek 1080p panel has:

        dsi_novatek_auo_1080_vid: somc,novatek_auo_1080p_video_panel {
                qcom,mdss-dsi-panel-name = "auo novatek 1080p vid";
                qcom,mdss-dsi-panel-controller = <&mdss_dsi0>;
                qcom,mdss-dsi-panel-type = "dsi_video_mode";
                qcom,mdss-dsi-panel-destination = "display_1";
                qcom,mdss-dsi-panel-framerate = <60>;
                qcom,mdss-dsi-virtual-channel-id = <0>;
                qcom,mdss-dsi-stream = <0>;
                qcom,mdss-dsi-panel-width = <1080>;
                qcom,mdss-dsi-panel-height = <1920>;
                qcom,mdss-dsi-h-front-porch = <56>;
                qcom,mdss-dsi-h-back-porch = <8>;
                qcom,mdss-dsi-h-pulse-width = <8>;
                qcom,mdss-dsi-h-sync-skew = <0>;
                qcom,mdss-dsi-v-back-porch = <8>;
                qcom,mdss-dsi-v-front-porch = <233>;
                qcom,mdss-dsi-v-pulse-width = <2>;
                qcom,mdss-dsi-h-left-border = <0>;
                qcom,mdss-dsi-h-right-border = <0>;
                qcom,mdss-dsi-v-top-border = <0>;
                qcom,mdss-dsi-v-bottom-border = <0>;
                qcom,mdss-dsi-bpp = <24>;
                qcom,mdss-dsi-underflow-color = <0x0>;
                qcom,mdss-dsi-border-color = <0>;
                somc,mdss-dsi-init-command = [15 01 00 00 00 00 02 FF E0
                                15 01 00 00 00 00 02 FB 01
                                15 01 00 00 00 00 02 B5 86
                                15 01 00 00 00 00 02 B6 77
                                15 01 00 00 00 00 02 B8 AD
                                15 01 00 00 00 00 02 FF 20
                                15 01 00 00 00 00 02 FB 01
                                15 01 00 00 00 00 02 10 04
                                15 01 00 00 00 00 02 FF 24
                                15 01 00 00 00 00 02 FB 01
                                15 01 00 00 00 00 02 C6 00
                                15 01 00 00 00 00 02 C5 32
                                15 01 00 00 00 00 02 92 92
                                15 01 00 00 00 00 02 FF 10
                                15 01 00 00 00 00 02 35 00
                                39 01 00 00 00 00 03 44 03 00
                                39 01 00 00 00 00 04 3B 03 30 06
                                15 01 00 00 01 00 02 BB 10
                                05 01 00 00 1E 00 01 11];
                qcom,mdss-dsi-on-command = [05 01 00 00 28 00 01 29];
                qcom,mdss-dsi-off-command = [15 01 00 00 00 00 02 FF 10
                                05 01 00 00 00 00 01 28
                                05 01 00 00 64 00 01 10];
                qcom,mdss-dsi-on-command-state = "dsi_lp_mode";
                qcom,mdss-dsi-off-command-state = "dsi_hs_mode";

                qcom,mdss-dsi-h-sync-pulse = <1>;
                qcom,mdss-dsi-traffic-mode = "non_burst_sync_event";
                qcom,mdss-dsi-bllp-eof-power-mode;
                qcom,mdss-dsi-bllp-power-mode;
                qcom,mdss-dsi-lane-0-state;
                qcom,mdss-dsi-lane-1-state;
                qcom,mdss-dsi-lane-2-state;
                qcom,mdss-dsi-lane-3-state;
                qcom,mdss-dsi-panel-timings = [E6 38 26 00 68 6E 2A 3C 2C 03 04 00];
                qcom,mdss-dsi-lp11-init;
                qcom,mdss-dsi-t-clk-post = <0x21>;
                qcom,mdss-dsi-t-clk-pre = <0x2B>;
                qcom,mdss-dsi-bl-min-level = <1>;
                qcom,mdss-dsi-bl-max-level = <4095>;
                qcom,mdss-brightness-max-level = <4095>;
                qcom,mdss-dsi-dma-trigger = "trigger_sw";
                qcom,mdss-dsi-mdp-trigger = "none";
                qcom,mdss-dsi-bl-pmic-control-type = "bl_ctrl_wled";
                qcom,mdss-dsi-pan-enable-dynamic-fps;
                qcom,mdss-dsi-pan-fps-update = "dfps_suspend_resume_mode";
                qcom,cont-splash-enabled;

                somc,driver-ic = <0>;
                somc,touch-reset-gpio = <&msmgpio 85 0>;
                somc,mdss-phy-size-mm = <64 114>;
                somc,mdss-dsi-lane-config = [00 c2 ef 00 00 00 00 01 ff
                                00 c2 ef 00 00 00 00 01 ff
                                00 c2 ef 00 00 00 00 01 ff
                                00 c2 ef 00 00 00 00 01 ff
                                00 02 45 00 00 00 00 01 97];
                somc,mdss-dsi-disp-on-in-hs = <1>;
                somc,mdss-dsi-wait-time-before-on-cmd = <150>;
                somc,lcd-id = <1>;
                somc,lcd-id-adc = <1236000 1395000>;
                somc,disp-en-on-post = <20>;
                somc,pw-on-rst-seq = <1 10>;
                somc,disp-en-off-post = <70>;
                somc,pw-off-rst-seq = <0 0>;
                somc,pw-down-period = <100>;

                somc,mdss-dsi-pre-uv-command = [23 01 00 00 00 00 02 B0 04];
                somc,mdss-dsi-uv-command = [06 01 00 00 00 00 01 DA
                        06 01 00 00 00 00 01 DB];
                somc,mdss-dsi-uv-param-type = <4>;
                somc,mdss-dsi-pcc-table-size = <225>;
                somc,mdss-dsi-pcc-table = <
                        0x00 0x01 0x38 0x3B 0x38 0x3B 0x5700 0x7000 0x8000
                        ... big blob ...
                        0xFF 0x00 0x00 0x3B 0x00 0x3B 0x8000 0x8000 0x8000>;
        };

The upstream kernel, by comparison, has the common panel framework (see `drivers/gpu/drm/panel`).  The common panel framework allows for panel drivers to be written once, and shared between different SoC's, so it uses drm's `struct mipi_dsi_device` to abstract away the display driver implementation.  For simple panels that don't require custom programming sequences there is `panel-simple.c`, but that is unlikely to be applicable for the DSI panel in any phone/tablet.  Instead, for panels requiring custom programming sequences, a new panel driver is written.  For this document, I will use the example of the `panel-auo-novatek-1080p-vid.c` driver that I am writing for the AUO panel in my xperia z3.  (Note that sony multi-sources their panel drivers so not all z3's use the AUO panel.)


