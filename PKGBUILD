# Submitter:   Wessel Dirksen "p-we" <wdirksen at gmail dot com>
# Contributor: Tycho LÃ1⁄4rsen "bas-t" (responsible for hosting and development of FFdecsawrapper)
# Contributor: Petr Vacek "vaca" (providing cardslot.conf for serial port SC readers)
# Contributor: J.P. van Best (implementing new procfs API for kernels >= 3.10 in FFdecsawrapper kernel module)
# Contributor: "Sunday" (tweak for speeding up gzip compression)

# !! Note: This package installs the TBS drivers for you. They should not be pre-installed

pkgname=ffdecsawrapper-git-tbs
pkgver=v150429
pkgrel=1
pkgdesc="FFdecsa empowered softcam - compiled with TBS DVB drivers + firmware"
url="https://github.com/bas-t/ffdecsawrapper.git"
arch=('i686' 'x86_64')
license=('GPLv3')
depends=('v4l-utils')
makedepends=('git' 'linux-headers')
optdepends=('oscam-svn: smartcard reader support' 'linuxtv-dvb-apps: handy DVB tools')
conflicts=('ffdecsawrapper' 'sasc-ng' 'tbs-dvb-drivers' 'tbs-linux-drivers')
provides=('ffdecsawrapper')
backup=('etc/camdir/cardclient.conf' 'etc/conf.d/ffdecsawrapper' 'etc/camdir/cardslot.conf')
install='ffdecsawrapper.install'

_tbsver=v150429

# !! Alternately you can use "oldstable" or "master" branch by changing line below

source=('git://github.com/bas-t/ffdecsawrapper.git#branch=stable' \
	"http://www.tbsdtv.com/download/document/common/tbs-linux-drivers_$_tbsver.zip" \
	'cardclient.conf' 'cardslot.conf' 'ffdecsawrapper.conf' \
	'ffdecsawrapper.install' 'ffdecsawrapper.lr' 'ffdecsawrapper.service' \
	'ffdecsawrapper.rc' 'tbs-mutex2.patch')
	 
sha256sums=('SKIP'
            'fdc905866a01231595e23c53b7b7b5e81428c10844215c1be1231c4a1297f743'
            '5c23db2b93d1accdc0b3f1612766de38bf7ede5658f6ef973706988dd71d1b81'
            '436eb5a612aa3cb9e45bb2031429f3d41eb596ed65d18659d3bd708919c61253'
            '3e4c28a68d312783761150c9bc5e8239261f9fba11f76f1ab8d072b71391c55c'
            'd6ceef7a559ad49722663b4f0b2762ff1ebce3801dd56cd801c24112c9a0cba9'
            'f435344dc9f1c0ed7c2e0de74ec434cd73e2130a0d7589a4d38338e45925d8db'
            'e798aacd050c078083a477b1fc393fa2dacf9b413ab8fea5a80449d3423c2b22'
            '1dcc2a7002805f09f8f2e582102bc5f3159d0d34d73c49b433872694ed633e8c'
            '93adfa959706848c5ca44e5c15da3539ff3b02082a72f912a94804bf3676fb28')

pkgver() {

	cd $srcdir/ffdecsawrapper
	_gitffdecsawrapper=`git describe --always | sed 's|-|.|g'`
	_kernel=`uname -r | sed -r 's/-/_/g'`
	echo "$_gitffdecsawrapper"_"$_tbsver"_"$_kernel"
}

prepare() {

	cd $srcdir
	rm -rf /linux-tbs-drivers
	tar xjvf linux-tbs-drivers.tar.bz2
	chmod 777 -R $srcdir
	cd $srcdir/linux-tbs-drivers

		if [ `uname -m` == "x86_64" ]; then
			./v4l/tbs-x86_64.sh  
		else
			./v4l/tbs-x86_r3.sh
		fi

	msg "Applying tbs-mutex-patch..."
	patch -p1 < $srcdir/tbs-mutex2.patch
	sleep 4
}

build() {

	cd $srcdir/linux-tbs-drivers
	make

	cd $srcdir/ffdecsawrapper      
	./configure --dvb_dir=$srcdir/linux-tbs-drivers/linux --update=no
}

package() {

	mkdir -p $pkgdir/usr/bin
	mkdir -p $pkgdir/usr/lib/modules/`uname -r`/updates/{tbs,ffdecsawrapper}
	mkdir -p $pkgdir/usr/lib/firmware
	mkdir -p $pkgdir/etc/conf.d
	mkdir -p $pkgdir/etc/rc.d
	mkdir -p $pkgdir/etc/camdir
	mkdir -p $pkgdir/etc/logrotate.d
	mkdir -p $pkgdir/usr/lib/systemd/system

	install -m0644 $srcdir/cardclient.conf  $pkgdir/etc/camdir/cardclient.conf
	install -m0644 $srcdir/cardslot.conf  $pkgdir/etc/camdir/cardslot.conf
	install -m0755 $srcdir/ffdecsawrapper.rc  $pkgdir/etc/rc.d/ffdecsawrapper
	install -m0644 $srcdir/ffdecsawrapper.conf  $pkgdir/etc/conf.d/ffdecsawrapper
	install -m0644 $srcdir/ffdecsawrapper.lr  $pkgdir/etc/logrotate.d/ffdecsawrapper-git-tbs.lr
	install -m0644 $srcdir/ffdecsawrapper.service  $pkgdir/usr/lib/systemd/system/ffdecsawrapper.service      

	install -m0755 $srcdir/ffdecsawrapper/ffdecsawrapper  $pkgdir/usr/bin
	install -m0755 $srcdir/ffdecsawrapper/dvbloopback.ko  $pkgdir/usr/lib/modules/`uname -r`/updates/ffdecsawrapper
	install -m0644 $srcdir/*dvb*.fw  $pkgdir/usr/lib/firmware
	find "$srcdir/linux-tbs-drivers" -name '*.ko' -exec cp {} $pkgdir/usr/lib/modules/`uname -r`/updates/tbs \;

	msg "Compressing modules, this will take awhile..."
	find "$pkgdir" -name '*.ko' -print0 | xargs -0 -P`nproc` -n10 gzip -9

	chmod 755 -R $pkgdir/usr/lib/modules/`uname -r`/updates
}

