genrule(
    name = "generate_libmodsecurity_config_h",
    srcs = ["@libmodsecurity//:autogen.sh"],
    outs = ["config.h"],
    cmd = """
        cp -R $$(dirname $(location @libmodsecurity//:autogen.sh)) modsec_src
        cd modsec_src
        ./autogen.sh
        ./configure --disable-shared --enable-static --with-pcre2 --with-lmdb --with-libxml --with-lua --with-curl=/usr/bin/curl-config --with-yajl --enable-doxygen-html --enable-doxygen-pdf

        cp src/config.h $@
    """,
)


cc_library(
    name = "modsecurity",
    srcs = ["@libmodsecurity//:modsecurity_sources"],
    hdrs = [":generate_libmodsecurity_config_h"],
    includes = ["src", "headers"],
    visibility = ["//visibility:public"],
)

genrule(
    name = "deps",
    srcs = [
        "@uthash//:include/utarray.h",
        "@uthash//:include/uthash.h",
        "@uthash//:include/utlist.h",
        "@uthash//:include/utringbuffer.h",
        "@uthash//:include/utstack.h",
        "@uthash//:include/utstring.h",

        "@libsodium//:libsodium",
        "@libmodsecurity//:modsecurity_sources",
        "@libcjson//:cjson",
        "@libcjson//:cJSON.h",
    ],
    outs = ["deps.tar.gz"],
    cmd = """
        mkdir -p deps/libsodium/include deps/libsodium/lib
        mkdir -p deps/uthash/include
        mkdir -p deps/libmodsecurity/include deps/libmodsecurity/lib
        mkdir -p deps/libcjson/include deps/libcjson/lib

        cp -L $(location @uthash//:include/utarray.h) deps/uthash/include
        cp -L $(location @uthash//:include/uthash.h) deps/uthash/include
        cp -L $(location @uthash//:include/utlist.h) deps/uthash/include
        cp -L $(location @uthash//:include/utringbuffer.h) deps/uthash/include
        cp -L $(location @uthash//:include/utstack.h) deps/uthash/include
        cp -L $(location @uthash//:include/utstring.h) deps/uthash/include

        libsodium_base=$$(dirname $$(echo '$(locations @libsodium//:libsodium)' | awk '{print $$1}'))
        cp -R $$libsodium_base/include/* deps/libsodium/include || true
        cp -R $$libsodium_base/lib/* deps/libsodium/lib || true

        libmodsecurity_base=$$(dirname $$(echo '$(locations @libmodsecurity//:modsecurity_sources)' | awk '{print $$1}'))
        cp -R $$libmodsecurity_base/include/* deps/libmodsecurity/include || true
        cp -R $$libmodsecurity_base/lib/* deps/libmodsecurity/lib || true

        cp -L $(location @libcjson//:cJSON.h) deps/libcjson/include

        for f in $(locations @libcjson//:cjson); do
            cp -L "$$f" deps/libcjson/lib
        done

        find deps -type d -exec chmod 755 {} +
        find deps -type f -exec chmod 644 {} +

        rm -f $(RULEDIR)/deps.tar.gz
        tar -zcvf $(RULEDIR)/deps.tar.gz deps
        rm -rf deps
    """,
)
