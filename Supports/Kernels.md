## 由咲音团队编译的内核文件 | Kernel files compiled by Sakion Team
| Device | Kernel android version | Kernel version | Re:Kernel version | Link | Source code | Binder | Signal |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| GKI | VANILLA_ICE_CREAM | 6.6.x | 7.5 | [Download](https://github.com/Sakion-Team/Re-Kernel/releases/tag/releases) | [Link](https://github.com/Sakion-Team/Re-Kernel/tree/main/LKM-Source) | ✔ | ✔ |
| GKI | UPSIDE_DOWN_CAKE | 6.1.x | 7.5 | [Download](https://github.com/Sakion-Team/Re-Kernel/releases/tag/releases) | [Link](https://github.com/Sakion-Team/Re-Kernel/tree/main/LKM-Source) | ✔ | ✔ |
| GKI | TIRAMISU | 5.15.x | 7.5 | [Download](https://github.com/Sakion-Team/Re-Kernel/releases/tag/releases) | [Link](https://github.com/Sakion-Team/Re-Kernel/tree/main/LKM-Source) | ✔ | ✔ |
| GKI | Android S | 5.10.x | 7.5 | [Download](https://github.com/Sakion-Team/Re-Kernel/releases/tag/releases) | [Link](https://github.com/Sakion-Team/Re-Kernel/tree/main/LKM-Source) | ✔ | ✔ |
## 由第三方开发者编译的内核文件 | Kernel files compiled by third-party developers
### 警告: 内核被收录并不代表内核源代码受到咲音团队审查，出现任何问题由用户自行承担。
### WARNING: The inclusion of the kernel does not mean that the kernel source code has been reviewed by the Sakion Team, any issues arising shall be borne by the user.
| Device | Kernel android version | Kernel version | Re:Kernel version | Link | Source code | Binder | Signal |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| Kernel Patch | N/A | 4.4-6.1 | 7.5 | [Download](https://github.com/lzghzr/APatch_kpm/releases/download/2024081800/re_kernel_6.0.10_network.kpm) | [Link](https://github.com/lzghzr/APatch_kpm/tree/main/re_kernel) | ✔ | ✔ |

## 内核兼容性追踪 | Kernel compatibility tracking
(✘) - 可能永远不会兼容 | Maybe never be compatible

(?) - 未知 | Unknown

(✔) - 兼容 | Compatible
| Device | Kernel android version | Kernel | Kernel version | Developer | Compatibility | Link | Kernel source code |
| :-: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| GKI | VANILLA_ICE_CREAM | ETO | 6.6.x | [ETO-妖刀](http://www.coolapk.com/u/2749672) | (✔) | N/A | N/A |
| GKI | UPSIDE_DOWN_CAKE | ETO | 6.1.x | [ETO-妖刀](http://www.coolapk.com/u/2749672) | (✔) | N/A | N/A |
| GKI | TIRAMISU | ETO | 5.15.x | [ETO-妖刀](http://www.coolapk.com/u/2749672) | (✔) | N/A | N/A |
| GKI | Android S | ETO | 5.10.x | [ETO-妖刀](http://www.coolapk.com/u/2749672) | (✔) | N/A | N/A |
| GKI | Android S | Marisa | 5.10.x | [Laulan56](https://github.com/Laulan56) | (✓) VER ≥ R5 | [Link](https://gitea.com/Laulan56/MarisaKernel_Marble-release) | N/A |
| GKI | Android S | Pixel Kernel | 5.10.x | Google Team | (✘) | N/A | N/A |
| GKI | Android S | VIVO Kernel | 5.10.x - 6.6.x | VIVO TEAM | (✘) | N/A | N/A |
| GKI | Android S | HarmonyOS Kernel | 5.10.x | HUAWEI TEAM | (✘) | N/A | N/A |
| GKI | TIRAMISU | ZTC | 5.15.x | [ztc1997](https://github.com/ztc1997) | (✔) | N/A | [Link](https://github.com/ztc1997/android_gki_kernel_5.15_common) |
| GKI | Android S | ZTC | 5.10.x | [ztc1997](https://github.com/ztc1997) | (✔) | N/A | [Link](https://github.com/ztc1997/android_gki_kernel_5.10_common) |
| GKI | VANILLA_ICE_CREAM | Haruhi | 6.6.x | [hamjin](https://github.com/hamjin) | (✔) | [Link](https://t.me/pandora_kernel_release) | N/A |
| GKI | UPSIDE_DOWN_CAKE | AngelBeats | 6.1.x | [hamjin](https://github.com/hamjin) | (✔) | [Link](https://t.me/pandora_kernel_release) | N/A |
| GKI | TIRAMISU | Yuni | 5.15.x | [hamjin](https://github.com/hamjin) | (✔) | [Link](https://t.me/pandora_kernel_release) | N/A |
| GKI | Android S | Pandora | 5.10.x | [hamjin](https://github.com/hamjin) | (✔) VER > 24.03.30 | [Link](https://t.me/pandora_kernel_release) | N/A |
| GKI | UPSIDE_DOWN_CAKE | Voyager | 6.1.x | [The Voyager](https://github.com/TheVoyager0777) | (✔) | N/A | N/A |
| GKI | TIRAMISU | Voyager | 5.15.x | [The Voyager](https://github.com/TheVoyager0777) | (✔) | N/A | N/A |
| GKI | Android S | Voyager | 5.10.x | [The Voyager](https://github.com/TheVoyager0777) | (✔) | N/A | N/A |
| ONLY XIAOMI/REDMI | N/A | Voyager | 4.x | [The Voyager](https://github.com/TheVoyager0777) | (✔) | N/A | N/A |

如果你的GKI内核与Re-Kernel不兼容 说明开发者对内核进行了破坏性修改

若你正在使用与Re-Kernel不兼容的GKI内核 你应当与内核开发者进行联系 而不是Re-Kernel开发者 或者使用官方内核而不是第三方内核
