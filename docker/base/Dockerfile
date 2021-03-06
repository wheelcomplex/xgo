# Go cross compiler (xgo): Base cross-compilation layer
# Copyright (c) 2014 Péter Szilágyi. All rights reserved.
#
# Released under the MIT license.

FROM ubuntu:14.04

MAINTAINER Péter Szilágyi <peterke@gmail.com>

# Create a small script to download binaries and validate their checksum
ENV FETCH ./fetch.sh
RUN \
  echo '#!/bin/bash'                   > $FETCH && \
  echo 'set -e'                       >> $FETCH && \
  echo 'file=`basename $1`'           >> $FETCH && \
  echo 'echo "Downloading $1..."'     >> $FETCH && \
  echo 'wget -q $1'                   >> $FETCH && \
  echo 'echo "$2  $file" > $file.sum' >> $FETCH && \
  echo 'sha1sum -c $file.sum'         >> $FETCH && \
  echo 'rm $file.sum'                 >> $FETCH && \
  chmod +x $FETCH


# Make sure apt-get is up to date and dependent packages are installed
RUN \
  apt-get update && \
  apt-get install -y automake autogen build-essential ca-certificates \
    gcc-arm-linux-gnueabi libc6-dev-armel-cross gcc-multilib gcc-mingw-w64 \
    clang llvm-dev  libtool libxml2-dev uuid-dev libssl-dev pkg-config \
    patch make xz-utils cpio wget unzip git mercurial --no-install-recommends

# Configure the container for OSX cross compilation
ENV OSX_SDK_PATH https://github.com/trevd/android_platform_build2/raw/master/osxsdks10.6.tar.gz
ENV OSX_SDK      MacOSX10.6.sdk

RUN \
  git clone https://github.com/tpoechtrager/osxcross.git && \
  sed -i.bak s/read/#read/g /osxcross/build.sh && \
  \
  $FETCH $OSX_SDK_PATH f526b4ae9806e8d31e3b094e3f004f8f160a3fad && \
  tar -xzf `basename $OSX_SDK_PATH` --strip-components 1 SDKs/$OSX_SDK && \
  tar -cjf /osxcross/tarballs/$OSX_SDK.tar.bz2 $OSX_SDK && \
  rm -rf `basename $OSX_SDK_PATH` $OSX_SDK && \
  \
  /osxcross/build.sh


# Create a small script to download all the Go packages, install the core Linux
# package, inject and bootstrap the others
ENV BOOTSTRAP ./bootstrap.sh
RUN \
  echo '#!/bin/bash'                                                                                        > $BOOTSTRAP && \
  echo 'set -e'                                                                                            >> $BOOTSTRAP && \
  echo                                                                                                     >> $BOOTSTRAP && \
  echo '$FETCH $DIST_LINUX_64  $DIST_LINUX_64_SHA1'                                                        >> $BOOTSTRAP && \
  echo '$FETCH $DIST_LINUX_32  $DIST_LINUX_32_SHA1'                                                        >> $BOOTSTRAP && \
  echo '$FETCH $DIST_LINUX_ARM $DIST_LINUX_ARM_SHA1'                                                       >> $BOOTSTRAP && \
  echo '$FETCH $DIST_OSX_64    $DIST_OSX_64_SHA1'                                                          >> $BOOTSTRAP && \
  echo '$FETCH $DIST_OSX_32    $DIST_OSX_32_SHA1'                                                          >> $BOOTSTRAP && \
  echo '$FETCH $DIST_WIN_64    $DIST_WIN_64_SHA1'                                                          >> $BOOTSTRAP && \
  echo '$FETCH $DIST_WIN_32    $DIST_WIN_32_SHA1'                                                          >> $BOOTSTRAP && \
  echo                                                                                                     >> $BOOTSTRAP && \
  echo 'tar -C /usr/local -xzf `basename $DIST_LINUX_64`'                                                  >> $BOOTSTRAP && \
  echo                                                                                                     >> $BOOTSTRAP && \
  echo 'tar -C /usr/local --wildcards -xzf `basename $DIST_LINUX_32` go/pkg/linux_386*'                    >> $BOOTSTRAP && \
  echo 'GOOS=linux GOARCH=386 /usr/local/go/pkg/tool/linux_amd64/dist bootstrap'                           >> $BOOTSTRAP && \
  echo 'tar -C /usr/local --wildcards -xzf `basename $DIST_LINUX_ARM` go/pkg/linux_arm*'                   >> $BOOTSTRAP && \
  echo 'GOOS=linux GOARCH=arm /usr/local/go/pkg/tool/linux_amd64/dist bootstrap'                           >> $BOOTSTRAP && \
  echo                                                                                                     >> $BOOTSTRAP && \
  echo 'tar -C /usr/local --wildcards -xzf `basename $DIST_OSX_64` go/pkg/darwin_amd64*'                   >> $BOOTSTRAP && \
  echo 'GOOS=darwin GOARCH=amd64 /usr/local/go/pkg/tool/linux_amd64/dist bootstrap'                        >> $BOOTSTRAP && \
  echo 'tar -C /usr/local --wildcards -xzf `basename $DIST_OSX_32` go/pkg/darwin_386*'                     >> $BOOTSTRAP && \
  echo 'GOOS=darwin GOARCH=386 /usr/local/go/pkg/tool/linux_amd64/dist bootstrap'                          >> $BOOTSTRAP && \
  echo                                                                                                     >> $BOOTSTRAP && \
  echo 'unzip -d /usr/local -q `basename $DIST_WIN_64` go/pkg/windows_amd64*'                              >> $BOOTSTRAP && \
  echo 'GOOS=windows GOARCH=amd64 /usr/local/go/pkg/tool/linux_amd64/dist bootstrap'                       >> $BOOTSTRAP && \
  echo 'unzip -d /usr/local -q `basename $DIST_WIN_32` go/pkg/windows_386*'                                >> $BOOTSTRAP && \
  echo 'GOOS=windows GOARCH=386 /usr/local/go/pkg/tool/linux_amd64/dist bootstrap'                         >> $BOOTSTRAP && \
  echo                                                                                                     >> $BOOTSTRAP && \
  echo 'rm -f `basename $DIST_LINUX_64` `basename $DIST_LINUX_32` `basename $DIST_LINUX_ARM` \\'           >> $BOOTSTRAP && \
  echo '  `basename $DIST_OSX_64` `basename $DIST_OSX_32` `basename $DIST_WIN_64` `basename $DIST_WIN_32`' >> $BOOTSTRAP && \
  chmod +x $BOOTSTRAP

ENV	PATH   /usr/local/go/bin:$PATH
ENV	GOPATH /go


# Create a small script to go get a package and cross compile it
ENV BUILD ./build.sh
RUN \
  echo '#!/bin/bash'                                                                            > $BUILD && \
  echo 'set -e'                                                                                >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Fetching $1...'                                                                   >> $BUILD && \
  echo 'go get $1'                                                                             >> $BUILD && \
  echo 'cd $GOPATH/src/$1'                                                                     >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'NAME=`basename $1`'                                                                    >> $BUILD && \
  echo 'if [ "$OUT" != "" ]; then'                                                             >> $BUILD && \
  echo '  NAME=$OUT'                                                                           >> $BUILD && \
  echo 'fi'                                                                                    >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'if [ "$FLAG_V" == "true" ]; then V=-v; fi'                                             >> $BUILD && \
  echo 'if [ "$FLAG_RACE" == "true" ]; then R=-race; fi'                                       >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Compiling for linux/amd64...'                                                     >> $BUILD && \
  echo 'GOOS=linux GOARCH=amd64 CGO_ENABLED=1 go build $V $R -o $NAME-linux-amd64$R'           >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Compiling for linux/386...'                                                       >> $BUILD && \
  echo 'GOOS=linux GOARCH=386 CGO_ENABLED=1 go build $V -o $NAME-linux-386'                    >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Compiling for linux/arm...'                                                       >> $BUILD && \
  echo 'CC=arm-linux-gnueabi-gcc \\'                                                           >> $BUILD && \
  echo '  GOOS=linux GOARCH=arm CGO_ENABLED=1 go build $V -o $NAME-linux-arm'                  >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Compiling for windows/amd64...'                                                   >> $BUILD && \
  echo 'CC=x86_64-w64-mingw32-gcc \\'                                                          >> $BUILD && \
  echo '  GOOS=windows GOARCH=amd64 CGO_ENABLED=1 go build $V $R -o $NAME-windows-amd64$R.exe' >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Compiling for windows/386...'                                                     >> $BUILD && \
  echo 'CC=i686-w64-mingw32-gcc \\'                                                            >> $BUILD && \
  echo '  GOOS=windows GOARCH=386 CGO_ENABLED=1 go build $V -o $NAME-windows-386.exe'          >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Compiling for darwin/amd64...'                                                    >> $BUILD && \
  echo '`/osxcross/target/bin/osxcross-env`'                                                   >> $BUILD && \
  echo 'CC=o64-clang \\'                                                                       >> $BUILD && \
  echo '  GOOS=darwin GOARCH=amd64 CGO_ENABLED=1 go build $V $R -o $NAME-darwin-amd64$R'       >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Compiling for darwin/386...'                                                      >> $BUILD && \
  echo 'CC=o32-clang \\'                                                                       >> $BUILD && \
  echo '  GOOS=darwin GOARCH=386 CGO_ENABLED=1 go build $V -o $NAME-darwin-386'                >> $BUILD && \
  echo                                                                                         >> $BUILD && \
  echo 'echo Moving binaries to host...'                                                       >> $BUILD && \
  echo 'cp `ls -t | head -n 7` /build'                                                         >> $BUILD && \
  chmod +x $BUILD

ENTRYPOINT ["./build.sh"]
