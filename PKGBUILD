# Maintainer: BlackEagle < ike DOT devolder AT gmail DOT com >
# Contributor: Paul Mattal <paul@archlinux.org>

pkgbase='lirc-bede'
pkgname='lirc-bede'
true && pkgname=('lirc-bede' 'lirc-bede-utils')
pkgver=0.9.0
pkgrel=34
arch=('i686' 'x86_64')
url="http://www.lirc.org/"
license=('GPL')
_extramodules=3.5-BEDE-external
makedepends=('linux-bede>=3.5' 'linux-bede<3.6' 'linux-bede-headers>=3.5' 'linux-bede-headers<3.6'
	'help2man' 'alsa-lib' 'libx11' 'libsm' 'python2' 'libftdi' 'libirman')
options=('!strip' '!makeflags')
source=(
	"http://downloads.sourceforge.net/lirc/lirc-$pkgver.tar.bz2"
	'lircd'
	'lircmd'
	'lirc.logrotate'
	'lircd.conf'
	'irexec.conf'
	'irexecd'
	'lirc_wpc8769l.patch'
	'lircd-handle-large-config.patch'
	'lirc_atiusb-kfifo.patch'
	'kernel-2.6.39.patch'
)
sha256sums=('6323afae6ad498d4369675f77ec3dbb680fe661bea586aa296e67f2e2daba4ff'
            '16e1285eb473a8e15796bc828dc11eb88a1007f30d2eb35727239a50859a7b34'
            '111f8bd4b69e7caa6d2b9da87938f949a5fcba01319c1611decbd51caf0ff497'
            'bd13ca00e30d85ff9166c03b8f7a20195ef89794e66d7e54f04ba1d014a73e7d'
            'a64962f310805db250e574464fce93cb4566a6f5bac1d9aad462435090bf3cb2'
            '398b9867c22537e71ac80e2e20a6915c52d90a8e542c1efaef6b8608a54e10ed'
            '175b4ce902688885b40423f2a55b3f55ccf54b55582f1fbd51a4cfcdc7a0cfb9'
            '137b1169810d1b66c5fe058aaffc2043ecbb4ef6cfce62050f9b418fa924b9ba'
            '474b5709e6604ef2815e6e1a611d77665e3d33be05cd09110330a81a846bc69f'
            'f2a83e2a32c8eb963453214d0337589a293b2327291290ec047f4d78782fb310'
            '3dddd4e9f093ee6fe75b3408da269744a4ffcd5255ea2382f077fb32079a2352')

build() {
	_kernver="$(cat /usr/lib/modules/$_extramodules/version)"

	cd lirc-$pkgver
	patch -Np1 -i "$srcdir/lirc_wpc8769l.patch"
	patch -Np1 -i "$srcdir/lircd-handle-large-config.patch"
	patch -Np1 -i "$srcdir/lirc_atiusb-kfifo.patch"
	patch -Np1 -i "$srcdir/kernel-2.6.39.patch"

	sed -i '/AC_PATH_XTRA/d' configure.ac
	sed -e 's/@X_CFLAGS@//g' \
		-e 's/@X_LIBS@//g' \
		-e 's/@X_PRE_LIBS@//g' \
		-e 's/@X_EXTRA_LIBS@//g' -i Makefile.am tools/Makefile.am

	libtoolize
	autoreconf

	PYTHON=python2 ./configure --enable-sandboxed \
		--prefix=/usr \
		--with-driver=all \
		--with-kerneldir=/usr/src/linux-$_kernver \
		--with-moduledir=/usr/lib/modules/$_kernver/kernel/drivers/misc \
		--with-transmitter

	# Remove drivers already in kernel
	sed -e "s:lirc_dev::" -e "s:lirc_bt829::" -e "s:lirc_igorplugusb::" \
		-e "s:lirc_imon::" -e "s:lirc_parallel::" -e "s:lirc_sasem::" \
		-e "s:lirc_serial::" -e "s:lirc_sir::" -e "s:lirc_ttusbir::" \
		-i Makefile drivers/Makefile drivers/*/Makefile tools/Makefile

	make
}

package_lirc-bede() {
	pkgdesc="Linux Infrared Remote Control kernel modules for BEDE kernel"
	depends=("lirc-bede-utils>=$pkgver" 'linux-bede>=3.5' 'linux-bede<3.6')
	install=lirc.install
	provides=('lirc')
	replaces=('lirc+pctv' 'lirc-bemm')

	cd lirc-$pkgver
	cd drivers
	make DESTDIR="$pkgdir" moduledir="/usr/lib/modules/$_extramodules/lirc" install

	# gzip all modules
	find "$pkgdir" -name '*.ko' -exec gzip -9  \;

	# set the kernel version in install script
	sed -i -e "s/EXTRAMODULES=.*/EXTRAMODULES=$_extramodules/g" \
		"$startdir/lirc.install"
}

package_lirc-bede-utils() {
	pkgdesc="Linux Infrared Remote Control utils for BEDE kernel"
	depends=('alsa-lib' 'libx11' 'libftdi' 'libirman')
	optdepends=('python2: pronto2lirc util')
	options=('strip' '!libtool')
	backup=('etc/conf.d/lircd.conf'
		'etc/conf.d/irexec.conf')
	provides=('lirc-utils')
	replaces=('lirc-bemm-utils')

	cd lirc-$pkgver
	make DESTDIR="$pkgdir" install
	install -d "$pkgdir/usr/share/lirc" "$pkgdir/etc/rc.d"
	cp "$srcdir"/{lircd,lircmd,irexecd} "$pkgdir/etc/rc.d"
	cp -rp remotes "$pkgdir/usr/share/lirc"
	chmod -R go-w "$pkgdir/usr/share/lirc/"

	# install the logrotate config
	install -Dm644 "$srcdir/lirc.logrotate" "$pkgdir/etc/logrotate.d/lirc"

	# install conf.d file
	install -Dm644 "$srcdir/lircd.conf" "$pkgdir/etc/conf.d/lircd.conf"

	# install conf.d file
	install -Dm644 "$srcdir/irexec.conf" "$pkgdir/etc/conf.d/irexec.conf"

	install -d "$pkgdir/etc/lirc"

	# remove built modules
	rm -r "$pkgdir/usr/lib/modules"
}
pkgdesc="Linux Infrared Remote Control kernel modules and utils for BEDE kernel"
