# ARMv5tel zaurus
# Maintainer: Yatao Li <me@yatao.info>
# Adapted from:
# - https://archlinuxarm.org/packages/armv7h/linux-armv7/files/PKGBUILD
# - https://github.com/greguu/voidz-packages/blob/voidz-packages-v03-(build8)/srcpkgs/linux5.0-zaurus/template

buildarch=1

pkgbase=linux
# doesn't patch: linux-5.15.39
# doesn't patch: linux-5.10.115
# doesn't compile: linux-5.4.193
# works: _srcname=linux-5.0
_srcname=linux-5.4.193
_kernelname=${pkgbase#linux}
_desc="ARMv5tel zaurus"
pkgver=5.4
pkgrel=2
arch=('arm')
url="http://www.kernel.org/"
license=('GPL2')
makedepends=('xmlto' 'kmod' 'inetutils' 'bc' 'git' 'libelf' 'cpio' 'perl' 'tar' 'xz')
options=('!strip')
source=("https://cdn.kernel.org/pub/linux/kernel/v5.x/${_srcname}.tar.xz"
        'config'
	'01_revert_mmc_slot-gpio.patch'
	'02_revert_mmc_spitz_pxamci.patch'
	'03_revert_mmc_pxamci_platform.patch'
	'04_revert_mmc_pxa_platform.patch'
	'05_revert_mmc_pxa_gpio_desc.patch'
	'06_revert_mmc_pxa_gpio_desc.patch'
	'07_revert_mmc_spi_gpio_desc.patch'
	'08_revert_pcmcia_cpufreq.patch'
	'09_usb_power.patch'
	'10_sandisk-cf.patch'
	'11-revert-mmc-slot-gpio.patch'
  '12-pcmcia-wifi-hack.patch'
  '01-fix-pxa27x-udc.patch'
)
md5sums=('SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
	 'SKIP'
   'SKIP'
 )

prepare() {
  cd "${srcdir}/${_srcname}"

  # add upstream patch
  # git apply --whitespace=nowarn ../patch-${pkgver}

  # apply greguu patches
  git apply ../01_revert_mmc_slot-gpio.patch
  git apply ../02_revert_mmc_spitz_pxamci.patch
  git apply ../03_revert_mmc_pxamci_platform.patch
  git apply ../04_revert_mmc_pxa_platform.patch
  git apply ../05_revert_mmc_pxa_gpio_desc.patch
  git apply ../06_revert_mmc_pxa_gpio_desc.patch
  git apply ../07_revert_mmc_spi_gpio_desc.patch
  git apply ../08_revert_pcmcia_cpufreq.patch
  git apply ../09_usb_power.patch
  git apply ../10_sandisk-cf.patch

  # apply yatli patches
  git apply ../11-revert-mmc-slot-gpio.patch
  git apply ../01-fix-pxa27x-udc.patch
  git apply ../12-pcmcia-wifi-hack.patch

  cat "${srcdir}/config" > ./.config

  # add pkgrel to extraversion
  sed -ri "s|^(EXTRAVERSION =)(.*)|\1 \2-${pkgrel}|" Makefile

  # don't run depmod on 'make install'. We'll do this ourselves in packaging
  sed -i '2iexit 0' scripts/depmod.sh
}

build() {
  cd "${srcdir}/${_srcname}"

  # get kernel version
  make prepare

  # load configuration
  # Configure the kernel. Replace the line below with one of your choice.
  #make menuconfig # CLI menu for configuration
  #make nconfig # new CLI menu for configuration
  #make xconfig # X-based configuration
  #make oldconfig # using old config from previous kernel version
  # ... or manually edit .config

  # Copy back our configuration (use with new kernel version)
  #cp ./.config ../${pkgbase}.config

  ####################
  # stop here
  # this is useful to configure the kernel
  #msg "Stopping build"
  #return 1
  ####################

  #yes "" | make config

  # build!
  make ${MAKEFLAGS} zImage modules
}

_package() {
  pkgdesc="The Linux Kernel and modules - ${_desc}"
  depends=('coreutils' 'linux-firmware' 'kmod' 'mkinitcpio>=0.7')
  optdepends=('crda: to set the correct wireless channels of your country')
  provides=("linux=${pkgver}" "WIREGUARD-MODULE")
  conflicts=('linux')

  cd "${srcdir}/${_srcname}"

  KARCH=arm

  # get kernel version
  _kernver="$(make kernelrelease)"
  _basekernel=${_kernver%%-*}
  _basekernel=${_basekernel%.*}

  mkdir -p "${pkgdir}"/{boot,usr/lib/modules}
  make INSTALL_MOD_PATH="${pkgdir}/usr" modules_install
  cp arch/$KARCH/boot/zImage "${pkgdir}/boot/zImage-${_kernver}"

  # add real version for building modules and running depmod from hook
  echo "${_kernver}" |
    install -Dm644 /dev/stdin "${pkgdir}/usr/lib/modules/${_extramodules}/version"

  # remove build and source links
  rm "${pkgdir}"/usr/lib/modules/${_kernver}/{source,build}

  # now we call depmod...
  depmod -b "${pkgdir}/usr" -F System.map "${_kernver}"

  # TODO setup kexec boot entry
}

_package-headers() {
  pkgdesc="Header files and scripts for building modules for linux kernel - ${_desc}"
  provides=("linux-headers=${pkgver}")
  conflicts=('linux-headers')

  cd ${_srcname}
  local _builddir="${pkgdir}/usr/lib/modules/${_kernver}/build"

  # Matching the VoidLinux template
  # L140, L142, L159
  install -Dt "${_builddir}" -m644 Makefile .config Module.symvers
  # L141
  install -Dt "${_builddir}/kernel" -m644 kernel/Makefile

  mkdir "${_builddir}/.tmp_versions"

  # L143, L147-152, L160
  cp -t "${_builddir}" -a include scripts

  # L1163
  install -Dt "${_builddir}/arch/${KARCH}" -m644 arch/${KARCH}/Makefile

  # L155-156
  cp -t "${_builddir}/arch/${KARCH}" -a arch/${KARCH}/include

  # Skip L164-171

  # L173-190
  # add headers for lirc package
  # pci
  for i in bt8xx cx88 saa7134; do
    mkdir -p ${_builddir}/drivers/media/pci/${i}
    cp -a drivers/media/pci/${i}/*.h ${_builddir}/drivers/media/pci/${i}
  done
  # usb
  for i in cpia2 em28xx pwc; do
    mkdir -p ${_builddir}/drivers/media/usb/${i}
    cp -a drivers/media/usb/${i}/*.h ${_builddir}/drivers/media/usb/${i}
  done
  # i2c
  mkdir -p ${_builddir}/drivers/media/i2c
  cp drivers/media/i2c/*.h ${_builddir}/drivers/media/i2c
  for i in cx25840; do
    mkdir -p ${_builddir}/drivers/media/i2c/${i}
    cp -a drivers/media/i2c/${i}/*.h ${_builddir}/drivers/media/i2c/${i}
  done

  # Skip L192-194

  # L197-198
  install -Dt "${_builddir}/drivers/md" -m644 drivers/md/*.h
  # L205-206
  install -Dt "${_builddir}/net/mac80211" -m644 net/mac80211/*.h

  # add xfs and shmem for aufs building
  # L228-231
  mkdir -p "${_builddir}"/{fs/xfs/libxfs,mm}
  cp fs/xfs/libxfs/xfs_sb.h ${_builddir}/fs/xfs/libxfs/xfs_sb.h

  # copy in Kconfig files
  # L241-245
  find . -name Kconfig\* -exec install -Dm644 {} "${_builddir}/{}" \;

  # remove unneeded architectures
  # L247-255
  local _arch
  for _arch in "${_builddir}"/arch/*/; do
    [[ ${_arch} == */${KARCH}/ ]] && continue
    rm -r "${_arch}"
  done

  # remove files already in linux-docs package
  rm -r "${_builddir}/Documentation"

  # remove now broken symlinks
  find -L "${_builddir}" -type l -printf 'Removing %P\n' -delete

  # Fix permissions
  chmod -R u=rwX,go=rX "${_builddir}"

  # strip scripts directory
  local _binary _strip
  while read -rd '' _binary; do
    case "$(file -bi "${_binary}")" in
      *application/x-sharedlib*)  _strip="${STRIP_SHARED}"   ;; # Libraries (.so)
      *application/x-archive*)    _strip="${STRIP_STATIC}"   ;; # Libraries (.a)
      *application/x-executable*) _strip="${STRIP_BINARIES}" ;; # Binaries
      *) continue ;;
    esac
    /usr/bin/strip ${_strip} "${_binary}"
  done < <(find "${_builddir}/scripts" -type f -perm -u+w -print0 2>/dev/null)
}

pkgname=("${pkgbase}" "${pkgbase}-headers")
for _p in ${pkgname[@]}; do
  eval "package_${_p}() {
    _package${_p#${pkgbase}}
  }"
done
