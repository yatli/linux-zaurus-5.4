# ARMv5tel zaurus
# Maintainer: Yatao Li <me@yatao.info>
# Adapted from:
# - https://archlinuxarm.org/packages/armv7h/linux-armv7/files/PKGBUILD
# - https://github.com/greguu/voidz-packages/blob/voidz-packages-v03-(build8)/srcpkgs/linux5.0-zaurus/template

buildarch=1

pkgbase=linux
# doesn't patch: linux-5.15.39
# doesn't patch: linux-5.10.115
# works: linux-5.4.193
# works: linux-5.0
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
  make ${MAKEFLAGS} zImage
}

