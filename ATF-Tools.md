

# OP-TEE 3.17.0 QEMU V8的环境搭建（Ubuntu20.04）

```bash
tools/cert_create/cert_create 
--tos-fw-extra1 /home/weston/workspace/optee-3.17/build/../optee_os/out/arm/core/tee-pager_v2.bin 
--tos-fw-extra2 /home/weston/workspace/optee-3.17/build/../optee_os/out/arm/core/tee-pageable_v2.bin

-n --tfw-nvctr 0 --ntfw-nvctr 0

--trusted-key-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/trusted_key.crt 
--key-alg rsa --key-size 2048 --hash-alg sha256 --rot-key /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/rot_key.pem

--tb-fw-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/tb_fw.crt
--tb-fw /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/bl2.bin 

--soc-fw-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/soc_fw_content.crt 
--soc-fw-key-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/soc_fw_key.crt
--soc-fw /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/bl31.bin 

--tos-fw-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/tos_fw_content.crt 
--tos-fw-key-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/tos_fw_key.crt
--tos-fw /home/weston/workspace/optee-3.17/build/../optee_os/out/arm/core/tee-header_v2.bin 

--nt-fw-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/nt_fw_content.crt 
--nt-fw-key-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/nt_fw_key.crt 
--nt-fw /home/weston/workspace/optee-3.17/build/../u-boot/u-boot.bin


tools/fiptool/fiptool create 
--tos-fw-extra1 /home/weston/workspace/optee-3.17/build/../optee_os/out/arm/core/tee-pager_v2.bin 
--tos-fw-extra2 /home/weston/workspace/optee-3.17/build/../optee_os/out/arm/core/tee-pageable_v2.bin

--trusted-key-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/trusted_key.crt

--tb-fw-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/tb_fw.crt
--tb-fw /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/bl2.bin

--soc-fw-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/soc_fw_content.crt 
--soc-fw-key-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/soc_fw_key.crt
--soc-fw /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/bl31.bin 

--tos-fw-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/tos_fw_content.crt 
--tos-fw-key-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/tos_fw_key.crt 
--tos-fw /home/weston/workspace/optee-3.17/build/../optee_os/out/arm/core/tee-header_v2.bin 

--nt-fw-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/nt_fw_content.crt 
--nt-fw-key-cert /home/weston/workspace/optee-3.17/trusted-firmware-a/build/qemu/release/nt_fw_key.crt 
--nt-fw /home/weston/workspace/optee-3.17/build/../u-boot/u-boot.bin


```



