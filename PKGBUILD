# Maintainer:  Linaro <maxim.anisimov.ua@gmail.com>
# Contributor: Chris Severance aur.severach aATt spamgourmet dott com
# Contributor: Lone_Wolf <lone_wolf@klaas-de-kat.nl>
# Contributor: Steven She <mintcoffee@gmail.com>
# Contributor: vbPadre <vbPadre@gmail.com>

pkgname='cnrdrvcups-sfp'

# The download link changes with every version, try to keep changes in one place
_pkgver='5.10';  _dl='0/0100005950/11'

pkgver="${_pkgver}"
pkgrel='1'
pkgdesc='Canon UFRII LT Printer Driver for Linux (LBP112/912, LBP113/913, LBP151dw, LBP6030/LBP6040/LBP6018L, LBP6230/LBP6240, LBP7100C/LBP7110C, LBP8100)'
arch=('x86_64' 'aarch64')
# Direct links to the download reference go bad on the next version. We want something that will persist for a while.
url='https://www.canon-europe.com'
license=('GPL2' 'MIT' 'custom')
# parts of the code are GPL or MIT licensed, some parts have a custom license
makedepends=(jbigkit gzip libxml2)
depends=(libcups glibc libstdc++ glib2 hicolor-icon-theme libjpeg6-turbo gtk3)
optdepends=('jbigkit: solves some cpu hangs'
            'ghostscript: necessary for printing on some devices'
            'at-spi2-core: for cnsetuputil2l'
            'gdk-pixbuf2: for cnsetuputil2l'
            'cairo: for cnsetuputil2l'
            'pango: for cnsetuputil2l'

)

conflicts=('cndrvcups-lb' 'cndrvcups-common-lb')
options=('!emptydirs' '!strip' '!libtool' '!lto')

source=(  "https://gdlp01.c-wss.com/gds/${_dl}/linux-UFRIILT-drv-v${_pkgver//\./}-m17n.tar.gz"
                replace_incorrect_int_with_char.patch
)
md5sums=('9f42743e4b030e8560163858711914f0'
         '8bc26ff46bf5877b5800b77685d5d917')
sha512sums=('4e00e2183d872a46f2da3c5b6cfb0fc24adb84266bfa9c03d939ee836a66e663898f539c572eacef368774abcf8594d51aea9712e730fd396754ef7e948b5e04'
            '1d118eeee1ce069b59db00cba5b534986ccbd1da3a9c4a4ba6892be4a478c2dac4bd83dae1b2dd28f0e58a145609c60940cd661fee87d025a12f856e161b1f65')

# Canon provides the sourcecode in a tarball within the dowload and we need to extract the code manually
# In order to keep the $srcdir structure tidy we put the extracted files in "extracted-${pkgname}-${_pkgver}" aka _srcdir
# the code itself is spread over many folders. 
# "cnrdrvcups-common-${_pkgver}" aka _common_dir & "cnrdrvcups-sfp-${_pkgver}" aka _driver_dir
# are used to keep this manageable

_srcdir="extracted-${pkgname}-${_pkgver}"
_common_dir="cnrdrvcups-common-${_pkgver}"
_driver_dir="cnrdrvcups-sfp-${_pkgver}"

prepare() {

    mkdir "${_srcdir}"
    cd "${_srcdir}"
    bsdtar -xf "${srcdir}/linux-UFRIILT-drv-v${_pkgver//\./}-m17n/Sources/${pkgname}-${pkgver}-1.00.tar.xz"

    for topatch in $(grep -rl cups.h . | grep '\.c'); do
      sed -i 's,#include <cups/cups.h>,#include <cups/ppd.h>,' $topatch
    done

    # Fix main.c:70:5: error: type of ‘mode’ defaults to ‘int’
    sed -i "s/int StartProcess(mode)/int StartProcess(int mode)/" cnrdrvcups-sfp-5.10/StatusMonitor/src/main.c

    local _specs=(cnrdrvcups-ncap.spec)

    # fix execjob.c:1154:108: error: passing argument 3 of 'add_param_int' makes integer from pointer without a cast
    patch --directory="${srcdir}"/$_srcdir/$_driver_dir/cngplp/cngplpmod/ --forward --input="$srcdir"/replace_incorrect_int_with_char.patch

    # the autogen.sh files from canon target an old automake/autoconf version
    # autoreconf converts them to a form compatible with archlinux autoconf/automake
    
    # Fix LDADD order in StatusMonitor
    sed -i 's/-lcups -lbuftool \..\/..\/cngplp\/cngplpmod\/libcngplpmod.la/..\/..\/cngplp\/cngplpmod\/libcngplpmod.la -lcups -lbuftool/' cnrdrvcups-sfp-5.10/StatusMonitor/src/Makefile.am

    pushd "${_common_dir}"
    for i in "backend" "buftool" "cngplp" "cnjbig" "rasterfilter"
    do
        pushd "$i"
        autoreconf --force --install --warnings=none
        popd
    done
    popd
    pushd "${_driver_dir}"
    for i in "cngplp/files" "cngplp" "cpca" "StatusMonitor"
    do
        pushd "$i"
        autoreconf --force --install --warnings=none
        popd
    done
    popd

    # allgen.sh where available is not useful for packaging on archlinux
    # Canon provides methods to build deb & rpm packages.
    # The debian rules are not suited for archlinux. When the .spec-file is converted to shell the resulting arch package works. 
    # Chris Severach figured out a way to automate  this conversion.

    # Generate make from spec %setup, %build
    sed -n -e '/^%setup/,/^%install/ p' "${_specs[@]}" | \
    grep -v '^%' | \
    sed -e '# Convert spec %{VAR} to shell ${VAR}' \
        -e 's:%{:${:g' \
        -e '# Quote to allow _cflags to have spaces' \
        -e 's:${_cflags}:"${_cflags}":g' \
        -e '# Some autogen.sh commands in the spec file do not set  --prefix. More than one --prefix dont cause problems so we can add it to all of them.' \
        -e 's:^./autogen.sh:& --prefix=${_prefix}:g' \
        > 'make.Arch'
     sed -i '1iset -e o pipefail' make.Arch

    # Generate make install from spec %install
    sed -n -e '/^%install/,/^%clean/ p' "${_specs[@]}" | \
    grep -v '^%' | \
    sed -e '# Convert spec %{VAR} to shell ${VAR}' \
        -e 's:%{:${:g' \
        -e '# Quote to handle path with spaces' \
        -e 's:${RPM_BUILD_ROOT}:"&":g' \
        -e 's:${LIBS}:${_libsarch}:g' \
        > 'make.install.Arch'
    sed -i '1iset -e o pipefail' make.install.Arch

}

_setvars() {
    # variables used by the (generated) make.Arch &  make.install.Arch files
    # relative paths start at ${srcdir}/${_srcdir} 
    # _libsarch is architecture dependent
    
    local -A _libsarchfolder
    _libsarchfolder['x86_64']="libs64/intel"
    _libsarchfolder['aarch64']="libs64/arm"

    _vars=(
        _builddir="${srcdir}/${_srcdir}"
        common_dir="${_common_dir}"
        driver_dir="${_driver_dir}"
        utility_dir="cnrdrvcups-utility-${_pkgver}"
        RPM_BUILD_DIR="${srcdir}/${_srcdir}"
        _prefix='/usr'
        _machine_type="MACHINETYPE="$CARCH
        _cflags="CFLAGS=$CFLAGS -fcommon -Wno-incompatible-pointer-types -Wno-int-conversion"
        _libdir='/usr/lib'
        _bindir='/usr/bin'
        libs32='/usr/lib'
        locallibs='/usr/lib/'
        _includedir='/usr/include'
        b_lib_dir="${srcdir}/${_srcdir}/lib"
        b_include_dir="${srcdir}/${_srcdir}/include"
        _libsarch="${_libsarchfolder[$CARCH]}"
        nobuild=0
  )
# -fcommon is needed to compile with gcc10 , see https://gcc.gnu.org/gcc-10/porting_to.html
# -O2 -pipe -fno-plt are taken from makepkg.conf default for archlinux
# _libsarch is architecture dependent
}

build() {
  
  set -e o pipefail
  cd "${_srcdir}"
  local _vars; _setvars
  # Bash does not recognize var assigments hidden by array expansion so we use env.
  env "${_vars[@]}" sh 'make.Arch'

}

package() {
    cd "${_srcdir}"

    local _vars; _setvars
    env "${_vars[@]}" \
    RPM_BUILD_ROOT="${pkgdir}" \
    sh 'make.install.Arch'

    # copy icons
    install -Dpm644 "cnrdrvcups-utility-${_pkgver}"/data/cnsetuputil.png "${pkgdir}"/usr/share/icons/hicolor/128x128/apps/cnsetuputil2l.png
    install -Dpm644 "cnrdrvcups-utility-${_pkgver}"/data/cngplp.png "${pkgdir}"/usr/share/icons/hicolor/128x128/apps/cngplp2l.png
    # copy .desktop files
    install -Dpm644 "cnrdrvcups-utility-${_pkgver}"/data/cnsetuputil2l.desktop "${pkgdir}"/usr/share/applications/cnsetuputil2l.desktop
    install -Dpm644 "cnrdrvcups-utility-${_pkgver}"/data/cngplp2l.desktop "${pkgdir}"/usr/share/applications/cngplp2l.desktop

    # licensing information is spread over multiple files and folders
    pushd "${_common_dir}"
    install -Dpm644 "README" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/README"
    
    install -Dpm644 "backend/LICENSE.txt" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/backend/LICENSE.txt"
    install -Dpm644 "backend/LICENSE.canon.txt" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/backend/LICENSE.canon.txt"
    install -Dpm644 "backend/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/backend/README"
    
    install -Dpm644 "buftool/LICENSE.txt" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/buftool/LICENSE.txt"
    install -Dpm644 "buftool/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/buftool/README"

    install -Dpm644 "cngplp/LICENSE.canon.txt" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/cngplp/LICENSE.canon.txt"
    install -Dpm644 "cngplp/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/cngplp/README"
    
    install -Dpm644 "cnjbig/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/cnjbig/README"
    
    install -Dpm644 "rasterfilter/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_common_dir}/rasterfilter/README"
    popd

    pushd "${_driver_dir}"
    install -Dpm644 "README" "${pkgdir}/usr/share/licenses/${pkgname}/${_driver_dir}/README"
    
    install -Dpm644 "cngplp/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_driver_dir}/cngplp/README"
    install -Dpm644 "cngplp/files/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_driver_dir}/cngplp/files/README"
    
    install -Dpm644 "cpca/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_driver_dir}/cpca/README"
    install -Dpm644 "cpca/cnpklib/LICENSE.canon.txt" "${pkgdir}/usr/share/licenses/${pkgname}/${_driver_dir}/cpca/cnpklib/LICENSE.canon.txt"
    
    install -Dpm644 "StatusMonitor/README" "${pkgdir}/usr/share/licenses/${pkgname}/${_driver_dir}/StatusMonitor/README"
    popd 
    
    # documentation
    pushd "$srcdir/linux-UFRIILT-drv-v${_pkgver//./}-m17n/Documents"
    
    install -Dpm644 deutsch/"README-ufr2lt-5.1xDE.html" "${pkgdir}/usr/share/doc/${pkgname}/README-ufr2lt-5.1xDE.html"
    install -Dpm644 espanol/"README-ufr2lt-5.1xSP.html" "${pkgdir}/usr/share/doc/${pkgname}/README-ufr2lt-5.1xSP.html"
    install -Dpm644 francais/"README-ufr2lt-5.1xFR.html" "${pkgdir}/usr/share/doc/${pkgname}/README-ufr2lt-5.1xFR.html"
    install -Dpm644 italiano/"README-ufr2lt-5.1xIT.html" "${pkgdir}/usr/share/doc/${pkgname}/README-ufr2lt-5.1xIT.html"
    install -Dpm644 uk_eng/"README-ufr2lt-5.1xUK.html" "${pkgdir}/usr/share/doc/${pkgname}/README-ufr2lt-5.1xUK.html"
    
    install -Dpm644 deutsch/"UsersGuide-ufr2lt-DE.html" "${pkgdir}/usr/share/doc/${pkgname}/UsersGuide-ufr2lt-DE.html"
    install -Dpm644 espanol/"UsersGuide-ufr2lt-SP.html" "${pkgdir}/usr/share/doc/${pkgname}/UsersGuide-ufr2lt-SP.html"
    install -Dpm644 francais/"UsersGuide-ufr2lt-FR.html" "${pkgdir}/usr/share/doc/${pkgname}/UsersGuide-ufr2lt-FR.html"
    install -Dpm644 italiano/"UsersGuide-ufr2lt-IT.html" "${pkgdir}/usr/share/doc/${pkgname}/UsersGuide-ufr2lt-IT.html"
    install -Dpm644 uk_eng/"UsersGuide-ufr2lt-UK.html" "${pkgdir}/usr/share/doc/${pkgname}/UsersGuide-ufr2lt-UK.html"
    
    install -Dpm644 deutsch/"LICENSE-DE.txt" "${pkgdir}/usr/share/licenses/${pkgname}/Documents/LICENSE-DE.txt"
    install -Dpm644 espanol/"LICENSE-ES.txt" "${pkgdir}/usr/share/licenses/${pkgname}/Documents/LICENSE-ES.txt"
    install -Dpm644 francais/"LICENSE-FR.txt" "${pkgdir}/usr/share/licenses/${pkgname}/Documents/LICENSE-FR.txt"
    install -Dpm644 italiano/"LICENSE-IT.txt" "${pkgdir}/usr/share/licenses/${pkgname}/Documents/LICENSE-IT.txt"
    install -Dpm644 uk_eng/"LICENSE-EN.txt" "${pkgdir}/usr/share/licenses/${pkgname}/Documents/LICENSE-EN.txt"
    popd
}
