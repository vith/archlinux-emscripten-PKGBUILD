# Maintainer: Sven-Hendrik Haase <svenstaro@gmail.com>
# Contributor: carstene1ns <arch carsten-teibes de> - http://git.io/ctPKG
# Contributor: Stefan Husmann <stefan-husmann@t-online.de>
# Contributor: Vlad Kolotvin <vlad.kolotvin@gmail.com>

pkgname=emscripten
pkgver=1.39.10
pkgrel=2
pkgdesc="LLVM-based project that compiles C and C++ into highly-optimizable JavaScript for the web"
arch=('x86_64')
url="http://emscripten.org"
license=('custom')
depends=(nodejs python binaryen)
makedepends=(cmake libxml2 git ninja)
optdepends=('java-environment: for using clojure'
            'ruby: for using websockify addon'
            'cmake: for emcc --show-ports')
install=emscripten.install
# Get commit SHAs from here:
# https://chromium.googlesource.com/emscripten-releases/+/refs/heads/master/DEPS
source=(emscripten-$pkgver.tar.gz::"https://github.com/kripken/emscripten/archive/$pkgver.tar.gz"
        git+https://github.com/llvm/llvm-project.git#commit=2150a6d0d635dea12c23dc84f356deeacbc8fbc2
        "emscripten.sh"
        arch-template.patch
        libcxxabi-include-libunwind.patch)
sha512sums=('00d8e0790f65f180271960bb59398294e7d37567c9f38b0b5a5ec3eb0f823c2d9a063cfa94e669a0445b2910e85a1fdeba85368813121dbe7844be4077ceae7f'
            'SKIP'
            'fbe9b95b8d18e7d0c6ec5fded6f11b72fbe4ddd0391e5704b281ba79c479f3563e82423b790ddf3f0554a23d659193ca898a81fe3db509f16c30c7188b790e4d'
            '04ffe0eac346d4accd54321aace952ccf3d6016243b98e3239de3fddc77c2c89ac4dfd66f65095c7f8a474e0e2b692bbbf3a150fde1dc410de920d5835f332a1'
            'b124ff6110810e3190bf05deda478c6fef044ff55a435df978fdb7ff7b4f312186add48cb99946b67a2467f7e28855e36606209c3c4dcee2898762ccc2e4c2ed')

prepare() {
  cd "$srcdir"/emscripten-$pkgver

  patch -Np1 --no-backup-if-mismatch -i "$srcdir"/arch-template.patch
  patch -Np1 --no-backup-if-mismatch -i "$srcdir"/libcxxabi-include-libunwind.patch

  mkdir "$srcdir"/llvm-project/llvm/build
}

build() {
  cd "$srcdir"/llvm-project/llvm/build

  # Inspired from https://github.com/WebAssembly/waterfall/blob/58e343a47ea02cb6daf19eefbdf626b23c24980c/src/build.py#L790
  cmake .. \
    -GNinja \
    -DPYTHON_EXECUTABLE=/usr/bin/python \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_SKIP_RPATH=YES \
    -DLLVM_TARGETS_TO_BUILD="X86;WebAssembly" \
    -DLLVM_BUILD_RUNTIME=OFF \
    -DLLVM_TOOL_LTO_BUILD=ON \
    -DLLVM_INSTALL_TOOLCHAIN_ONLY=ON \
    -DLLVM_INCLUDE_EXAMPLES=OFF \
    -DLLVM_INCLUDE_TESTS=OFF \
    -DLLVM_ENABLE_PROJECTS="lld;clang" \
    -DCLANG_INCLUDE_TESTS=OFF
  ninja

  cd "$srcdir"/emscripten-"$pkgver"
  npm ci
}

package() {
  # Install LLVM stuff according to https://github.com/emscripten-core/emscripten/blob/incoming/docs/process.md
  install -d "$pkgdir"/usr/lib
  cp -r "$srcdir"/llvm-project/llvm/build/bin "$pkgdir"/usr/lib/emscripten-llvm

  # Install emscripten
  cd "$srcdir"/emscripten-$pkgver
  install -d "$pkgdir"/usr/lib/emscripten
  cp -rup em* cmake site src system third_party tests tools package{,-lock}.json node_modules "$pkgdir"/usr/lib/emscripten

  # Remove clutter
  rm "$pkgdir"/usr/lib/emscripten/*.bat

  install -d "$pkgdir"/usr/share/doc
  ln -s /usr/lib/emscripten/site/source/docs "$pkgdir"/usr/share/doc/$pkgname
  install -Dm755 "$srcdir"/emscripten.sh "$pkgdir"/etc/profile.d/emscripten.sh
  install -Dm644 LICENSE "$pkgdir"/usr/share/licenses/$pkgname/LICENSE
}
