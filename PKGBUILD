# Maintainer: Llewelyn Trahaearn <WoefulDerelict [at] GMail [dot] com>
# Contributor: Oleksandr Natalenko <pfactum [at] gmail [dot] com>
# Contributor: Igor Mosyagin <c6h10o5 [at] gmail [dot] com>

pkgname=brother-mfc-7860dw
pkgver=2.1.0
_wrapper_ver=2.0.4
pkgrel=5
pkgdesc="LPR and CUPSwrapper driver for the Brother MFC-7860DW."
arch=('i686' 'x86_64')
url="http://support.brother.com/g/b/producttop.aspx?c=us_ot&lang=en&prod=mfc7860dw_all"
license=('GPL')
depends=('cups' 'ghostscript' 'gsfonts' 'cups-filters')
depends_x86_64=('lib32-glibc')
source=(http://www.brother.com/pub/bsc/linux/dlf/mfc7860dwlpr-${pkgver}-1.i386.rpm
        http://www.brother.com/pub/bsc/linux/dlf/cupswrapperMFC7860DW-${_wrapper_ver}-2.i386.rpm
        brother-mfc-7860dw.patch)
sha256sums=('7886fbbaa9aa980d6f6b08f4a584f8614857e6ab1511c2b494d3b8e1dedafd14'
            '2ab019790bd88b65b592b1a700f9e96018130bcbfc19cde5017f0230e7f77e98'
            '8025aafc6e4a80dc162500522bc351a5b07bdc81187bd0dd3de661468e5a3d10')

prepare() {
  # Keep the vendor's /usr/local paths and disable its system-changing installer behavior.
  patch -p1 < "${srcdir}/brother-mfc-7860dw.patch" || return 1
}

build() {
  sh "${srcdir}/usr/local/Brother/Printer/MFC7860DW/cupswrapper/cupswrapperMFC7860DW-${_wrapper_ver}"
}

package() {
  install -d "${pkgdir}/usr/local/Brother"
  cp -r "${srcdir}/usr/local/Brother/Printer" "${pkgdir}/usr/local/Brother/"
  cp -r "${srcdir}/var" "${pkgdir}/"

  local printer_dir="${pkgdir}/usr/local/Brother/Printer/MFC7860DW"
  local inf_dir="${printer_dir}/inf"
  local rc_file="${inf_dir}/brMFC7860DWrc"

  chgrp cups "${inf_dir}" "${rc_file}"
  chmod 2775 "${inf_dir}"
  chmod 664 "${rc_file}"

  rm "${printer_dir}/cupswrapper/cupswrapperMFC7860DW-${_wrapper_ver}"
  rm "${inf_dir}/setupPrintcap2"

  install -m 644 -D ppd_file "${pkgdir}/usr/share/cups/model/MFC7860DW.ppd"
  install -m 755 -D wrapper  "${pkgdir}/usr/lib/cups/filter/brlpdwrapperMFC7860DW"
}
