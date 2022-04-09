## 测试

新建默认wasm工程,非pwa,测试链接pwa名称请大家忽略.

| 方式 | URL | 发布后 | rar压缩包 | chrome 隐私模式 | edge隐私模式 | 备注 |
| :----: | :----: | :----: | :----: | :----: | :----: | :----: |
| WASM+BR |  | 15.5 m | 8.76 m | 1.87s | 2.09s | |
| WASM AOT |  | 32.5 m | 16.2 m | 3.75s | 2.8s | |
| WASM+BR (net7pre2) |  | 16.2 m | 9.05 m | 1.91s | 2.68s | net6工程升级 |
| WASM AOT (net7pre2)|  | 27.7 m | 14.6 m | 2.54s | 2.69s | net6工程升级 |
| WASM+BR (net7pre2) | https://testbr.app1.es | 16.2 m | 9.23 m | 1.89s | 1.99s | 新建工程 |
| WASM AOT (net7pre2)| https://testbrpwa.app1.es | 36.3 m | 17.3 m | 2.52s | 2.75s | 新建工程 |

## 服务器Nginx配置

运行nginx -V检查是否带br, 检查 module= `ngx_brotli` 关键字. 如果不带,自行编译安装.
```
nginx -V
nginx version: nginx/1.20.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-44) (GCC) 
built with OpenSSL 1.1.1k  25 Mar 2021
TLS SNI support enabled
configure arguments: --user=www --group=www --prefix=/www/server/nginx --add-module=/www/server/nginx/src/ngx_devel_kit --add-module=/www/server/nginx/src/lua_nginx_module --add-module=/usr/src/ngx_brotli --add-module=/www/server/nginx/src/ngx_cache_purge --add-module=/www/server/nginx/src/nginx-sticky-module --with-openssl=/www/server/nginx/src/openssl --with-pcre=pcre-8.43 --with-http_v2_module --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module --with-http_stub_status_module --with-http_ssl_module --with-http_image_filter_module --with-http_gzip_static_module --with-http_gunzip_module --with-ipv6 --with-http_sub_module --with-http_flv_module --with-http_addition_module --with-http_realip_module --with-http_mp4_module --with-ld-opt=-Wl,-E --with-cc-opt=-Wno-error --with-ld-opt=-ljemalloc --with-http_dav_module --add-module=/www/server/nginx/src/nginx-dav-ext-module
```

brotli 配置
```
        # brotli 配置开始
        brotli on;
        brotli_comp_level 6;  #压缩等级，默认6，最高11，太高的压缩水平可能需要更多的CPU
        brotli_buffers 16 8k;  #请求缓冲区的数量和大小
        brotli_min_length 100; #指定压缩数据的最小长度，只有大于或等于最小长度才会对其压缩。这里指定100字节
        brotli_types text/plain application/javascript application/x-javascript text/javascript text/css application/xml application/json image/svg application/font-woff application/vnd.ms-fontobject application/vnd.apple.mpegurl image/x-icon image/jpeg image/gif image/png image/bmp;  #指定允许进行压缩类型
        brotli_static always;   #是否允许查找预处理好的、以.br结尾的压缩文件，可选值为on、off、always
        brotli_window 512k;  #窗口值，默认值为512k
        proxy_set_header Accept-Encoding "";
        # brotli 配置结束
```

## 结论

PWA不能用import { BrotliDecode } from './decode.min.js'; 直接会导致pwa离线功能失效 , 就走nginx自己的br就够
空wasm [非pwa] 工程,用上面的js. 的确纯走br了

## 技巧

hash校验可以强制关掉或者自己再算一遍.  .map(asset => new Request(asset.url, {  cache: 'no-cache' }));

