# glog 0.6.0 on loongarch64

## 必须打 Doris 自带补丁

Doris 的 `thirdparty/patches/glog-0.6.0.patch` 给 glog 增加了两个配置项：
`FLAGS_log_split_method`、`FLAGS_log_filenum_quota`。不打这个补丁，编译 Doris BE 的 `common/logconfig.cpp` 会报：

    error: ‘FLAGS_log_filenum_quota’ was not declared in this scope
    error: ‘FLAGS_log_split_method’在此作用域中尚未声明

## 构建（按 Doris 官方参数）

    patch -p1 < doris-thirdparty/patches/glog-0.6.0.patch
    cmake -S . -B build -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX=$PREFIX \
          -DCMAKE_BUILD_TYPE=Release -DCMAKE_POSITION_INDEPENDENT_CODE=ON \
          -DWITH_UNWIND=OFF -DBUILD_SHARED_LIBS=OFF -DWITH_TLS=OFF
    cmake --build build -j4 && cmake --build build --target install

## 为什么必须 WITH_UNWIND=OFF

如果按默认（ON）构建，glog 会链接 libunwind，之后**每个用到 glog 的目标都要补 `-lunwind`**，否则报：

    undefined reference to `_Uloongarch64_getcontext'
    undefined reference to `_ULloongarch64_init_local'

Doris 官方就是 `-DWITH_UNWIND=OFF`。

（补充：本机系统自带 libunwind 1.8.2 且**带 loongarch64 后端**，符号齐全，Doris 的 `USE_UNWIND=ON` 其实也能靠系统库满足，只是 Doris 只去 thirdparty 目录找。）
