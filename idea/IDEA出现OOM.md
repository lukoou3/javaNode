
## idea build project OOM
idea java工程build OOM。

idea本身配置的内存不小，项目构建时，仍会出现OOM异常，这一般是由于idea限制了项目构建时的heap size导致的，可以按需将这个数值调大。

参数位置：setting > Build,Execution,Deployment > Compiler > Shared build process heap size(Mbytes)

默认好像是700改为2048后，编译通过。
