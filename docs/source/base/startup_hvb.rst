startup_hvb
=================================

overview
------------

OpenHarmony Verified Boot (HVB)

`startup_hvb <https://gitee.com/openharmony/startup_hvb>`_

hvb与android avb类似。

hvb的rvt分区与android vbmeta类似。

dm-verity基本相同。

适配
-------

bootloader是rvt pubkey、rollback index、unlock flag。

bootloader向os传参通过`hvb_creat_cmdline`生成。

kernel选项打开。

fstab参数打开。

data structure
-------------------

参考 `hvb.h`, `hvb_cert.h`, `hvb_footer.h`, `hvb_rvt.h`。

`test/hvb_main.c`
---------------------

校验入口在 `hvb_chain_verify` 。

适配 `hvb_ops.h` 相关函数。

适配 `hvb_sysdeps.h` 相关函数。

`auth/hvb.c`
---------------

1. `hvb_chain_verify`

    - `hvb_rvt_verify_root` 校验rvt分区。
    - `hvb_walk_verify_nodes` 基于rvt分区提供的子分区pubkey，校验各子分区。
    - `hvb_creat_cmdline` 生成命令行参数，其中包含已校验的分区digest，传递给OS。

#. `hvb_rvt_verify_root`

    - `footer_init_desc` 校验rvt分区的自签名，读取footer中包含的 `rvt_pubkey` 。
    - `ops->valid_rvt_key` 校验 `rvt_pubkey` 是否可信。应适配。

#. `hvb_walk_verify_nodes`

    - `hvb_rvt_get_pubk_buf` 从rvt分区读取指定子分区的 `expected_pubkey` 。
    - `footer_init_desc` 校验指定子分区的自签名，读取footer中包含子分区的 `pubkey` 。
    - `hvb_buf_equal` 比较 `expected_pubkey` 与子分区中的pubkey是否相等。

存在优化空间，先比较公钥，再验签。

`footer/hvb_footer.c`
----------------------

1. `footer_init_desc`

    - `_load_and_parse_footer` 从分区中读取footer。
    - `_load_cert` 从footer读取分区相关签名信息。
    - `cert_init_desc` 校验指定分区的自签名，读取该分区的 `pubkey` 。

`cert/hvb_cert.c`
-----------------------

1. `cert_init_desc`

    - `hvb_verify_cert` 校验分区自签名。
    - `_check_rollback_index` 防回滚。
    - 校验通过，写入pubkey至指定 `*out_pubk = sign_info->pubk;` 。

