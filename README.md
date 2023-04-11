# BTFHubForAndroid

We need BTFHub for Android too.

# Build Android Common Kernel

**build common-android12-5.10 kernel with BTF example**

sync kernel source code

```bash
git clone https://gerrit.googlesource.com/git-repo
mkdir android-kernel && cd android-kernel
../git-repo/repo init --depth=1 --u https://android.googlesource.com/kernel/manifest -b common-android12-5.10
../git-repo/repo sync -j$(nproc --all)
```

enable CONFIG_DEBUG_INFO_BTF

```bash
cd common
make ARCH=arm64 gki_defconfig
make ARCH=arm64 menuconfig
# plz enable CONFIG_DEBUG_INFO_BTF on GUI
make ARCH=arm64 savedefconfig
cp defconfig arch/arm64/configs/gki_defconfig
rm .config
cd ..
```

perpare toolchains

```bash
mkdir gcc-aosp
wget https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/+archive/refs/tags/android-12.1.0_r27.tar.gz
tar -C gcc-aosp/ -zxvf android-12.1.0_r27.tar.gz
```

setup env and build, note: clang version can be found at `build.config.common`

```bash
export PATH=/home/kali/Downloads/android-kernel/prebuilts-master/clang/host/linux-x86/clang-r416183b/bin:$PATH
make -j$(nproc --all) O=out ARCH=arm64 CLANG_TRIPLE=aarch64-linux-gnu- CROSS_COMPILE=/home/kali/Downloads/gcc-aosp/bin/aarch64-linux-android- CC=clang LD=ld.lld gki_defconfig
make -j$(nproc --all) O=out ARCH=arm64 CLANG_TRIPLE=aarch64-linux-gnu- CROSS_COMPILE=/home/kali/Downloads/gcc-aosp/bin/aarch64-linux-android- CC=clang LD=ld.lld
```

vmlinux is out here => `./out/vmlinux`

# Build Pahole

```bash
sudo apt install libdwarf-dev
sudo apt install libdw-dev
git clone https://github.com/acmel/dwarves --depth=1
cd dwarves
mkdir build
cd build
cmake -D__LIB=lib -DBUILD_SHARED_LIBS=OFF ..
sudo make install
```

# Pack vmlinux

```bash
pahole --btf_encode_detached a12-5.10-arm64.btf ./out/vmlinux
tar cvfJ "./a12-5.10-arm64.btf.tar.xz" a12-5.10-arm64.btf
```

# Links

- [KernelSU_Action build-kernel.yml](https://github.com/xiaoleGun/KernelSU_Action/blob/main/.github/workflows/build-kernel.yml)
- [KernelSU gki-kernel.yml](https://github.com/tiann/KernelSU/blob/main/.github/workflows/gki-kernel.yml)
- [btfhub update.sh](https://github.com/aquasecurity/btfhub/blob/main/tools/update.sh)