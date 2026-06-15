# 生物识别认证

## 定位

这是生物识别设备、指纹驱动、设备识别和认证服务的分类入口。`GW_Fingerprint_PA` 和 Pixelauth T350P 是当前已沉淀的具体线索。

## 适用场景

- 设置界面指纹状态断开。
- 已安装驱动但设备未被识别。
- 需要恢复或验证指纹驱动和服务。

## 先读知识章节

- 指纹驱动和生物识别具体诊断：[`../knowledge/hardware/biometric-fingerprint.md`](../knowledge/hardware/biometric-fingerprint.md)

## 最小诊断

```bash
systemctl status biometric-authentication.service --no-pager
lsusb
dpkg -l | rg -i 'finger|biometric|pixelauth|gw_' || true
```
