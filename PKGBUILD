# Maintainer: John Schoenick <john@pointysoftware.net>

pkgname=rtags
pkgver=2.0
_rtags_git_tag="v$pkgver"
pkgrel=1
pkgdesc='A client/server application that indexes C/C++ code and keeps a persistent in-memory database of references, declarations, definitions, symbolnames etc.'
arch=('i686' 'x86_64')
url='https://github.com/Andersbakken/rtags'
license=('GPL3')
makedepends=('git' 'cmake' 'emacs')
depends=('bash' 'clang')

# rct has no release tarballs, and the source tarballs are just auto-created
# from tags by github anyway, so we'll just use git as the source with specific
# tags.
# (See rtags-git AUR package for dynamic versioning)
source=("git+https://github.com/Andersbakken/rtags#tag=v${pkgver}"
        # Tag of rct we fetch doesn't matter, we'll use the rtags desired
        # submodule commit either way.
        "rtags-rct::git+https://github.com/Andersbakken/rct")
sha1sums=('SKIP'
          'SKIP')

prepare() {
  cd "${srcdir}/rtags"
  git config submodule.src/rct.url "${srcdir}"/rtags-rct
  git submodule update --init src/rct
}

build() {
  cd "${srcdir}"

  # Nuke old build dirs
  rm -rf build elisp
  mkdir build elisp

  cd build
  msg2 "Configuring with cmake"
  cmake "${srcdir}"/rtags -DCMAKE_INSTALL_PREFIX=/usr
  msg2 "Building"
  make

  msg2 "Compiling elisp files"
  cd "${srcdir}"
  cp -av rtags/src/*.el elisp

  # FIXME the rtags plugins for auto-complete and company mode wont compile if
  #       you don't have them installed, and there are no arch packages for them
  #       currently. (bytecode compilation is not required, but provides a
  #       speedup)
  for elispSrc in elisp/*.el; do
    if ! emacs -batch -f batch-byte-compile "$elispSrc"; then
      warning "Emacs library ${elispSrc} won't be byte-compile'd. Usually this means the required dependencies not installed."
    fi
  done
}

package() {
  cd "${srcdir}"/build
  make DESTDIR="${pkgdir}" install

  # Install emacs .el files
  cd "${srcdir}"/elisp
  install -d "${pkgdir}/usr/share/emacs/site-lisp"
  install -D -m644 *.{el,elc} "${pkgdir}/usr/share/emacs/site-lisp"

  # Misc wrapper/helper scripts not put in bin by make install go here
  install -d "${pkgdir}/usr/lib/rtags/bin"
  for misc in "${srcdir}"/build/bin/*; do
    if [[ ! -f "${pkgdir}/usr/bin/$(basename "$misc")" ]]; then
      install -D -m755 "$misc" "${pkgdir}"/usr/lib/rtags/bin
    fi
  done
}
