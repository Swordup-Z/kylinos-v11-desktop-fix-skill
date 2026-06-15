# 源码重编译

## 定位

这是需要修改系统组件源码、重新构建系统二进制或评估 ABI 风险的问题分类入口。

## 适用场景

- 系统设置界面或后端逻辑写死，配置级方案无法满足需求。
- 需要替换系统共享库、控制中心插件或系统服务。
- 需要评估源码版本、候选 git 节点、ABI/SONAME/依赖/RPATH 风险。
- 需要保存本地客制化源码、patch、构建/回滚临时目录和后续继续修改所需索引。

## 先读知识章节

- 源码重编译通用流程：[`../knowledge/source-rebuild/README.md`](../knowledge/source-rebuild/README.md)
- 本地源码客制化索引、`CUSTOMIZATION.md`、DATA 分区工作区、patch 保存和构建/回滚清理规则：[`../knowledge/source-rebuild/local-customization-index.md`](../knowledge/source-rebuild/local-customization-index.md)
- UKUI 全局搜索搜索引擎修改：[`../knowledge/source-rebuild/ukui-search-web-engine.md`](../knowledge/source-rebuild/ukui-search-web-engine.md)
- UKUI 全局搜索自定义命令 provider：[`../knowledge/source-rebuild/ukui-search-command-provider.md`](../knowledge/source-rebuild/ukui-search-command-provider.md)
- UKUI 系统托盘隐藏区稳定性源码级修改：[`../knowledge/source-rebuild/ukui-system-tray.md`](../knowledge/source-rebuild/ukui-system-tray.md)

## 最小诊断

```bash
dpkg-query -W -f='${binary:Package} ${Version} ${Source}\n' <package>
apt-cache policy <package>
apt-cache showsrc <source-package> 2>&1 | sed -n '1,80p'
test -d /data/usershare/kylinos-local-sources && find /data/usershare/kylinos-local-sources -maxdepth 2 -name CUSTOMIZATION.md -print
```
