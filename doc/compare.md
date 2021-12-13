# tern vs. orion

The following describes the differences between `tern` and `orion` in three scenarios. For each scenario we list the Dockerfile segment and the outputs produced by `tern` and `orion` respectively. By comparing the outputs, we draw the differences between the two tools which are listed under **Comparison** subsection for each scenario.

## Scenario 1. Copy/Add files

### Comparison

1. **tern** treats copied/added file as "Unknown content. Additional analysis may be required."
2. **tern** reports "Packages found in Layer: None".
3. **orion** recoganizes `COPY` and `ADD` operations.
4. **orion** reports COPY/ADD of a directory as a spdx package and lists all files in the directory
5. **orion** reports COPY/ADD of an individual file as a spdx file independent of packages.  

**Dockerfile segment**

```
COPY LICENSE.txt LICENSE.txt

```

**Outputs**

***By tern***

```
	Layer 4:
		info: Layer created by commands: /bin/sh -c #(nop) COPY file:34eb0cc1f4f8a1ac753cd4174fb4e2e9aa3604687050fe4b2b14b04f288f61d1 in LICENSE.txt 
		warning: Unknown content. Additional analysis may be required.
		info: Retrieved package metadata using apk default method. 

	File licenses found in Layer:  None
	Packages found in Layer: None
```

***By orion***

```
##### Files independent of packages

FileName: LICENSE.txt
SPDXID: SPDXRef-File-99079d639b55edf54a58db08ba641e499a4140e8
FileChecksum: SHA1: d4657158683ed259360621516745639f7d5c83c4
FileChecksum: SHA256: 72fc6d84e9b1a939bc69d03b7afeec0369a87a2c6c3044adff8707774ec30027
FileChecksum: SHA512: fd3d39036cc5d070f7cf493ddeddaea905ab8cd4c64613dc520c413d86e716037b7ce2e82efd509ced4233add245e5f5e4641364675f20c2e683ca3c5856f3bb
LicenseConcluded: NOASSERTION
LicenseInfoInFile: NOASSERTION
FileCopyrightText: NOASSERTION

Relationship: SPDXRef-DOCUMENT-FOR-ADDONS DESCRIBES SPDXRef-File-99079d639b55edf54a58db08ba641e499a4140e8
```

## Scenario 2. Install an executable that is not managed by package managers

### Comparison

1. **tern** unrecognizes `wget` command and reports "Unrecognized Commands:wget ...".
2. **tern** reports no packages or files with "Packages found in Layer: None".
3. **orion** recoganizes `wget` operation, treats it as a spdx package with the package download location.
4. **orion** lists all the files installed from the download and their relationship with the spdx package.

**Dockerfile segment**

```
RUN wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 &&\
    chmod +x ./jq &&\
    cp jq /usr/bin &&\
    jq --version
```

**Outputs**

***By tern***

```
Layer 3:
		info: Layer created by commands: /bin/sh -c wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 &&    chmod +x ./jq &&    cp jq /usr/bin &&    jq --version
		warning: 
Unrecognized Commands:wget -O jq https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
chmod +x ./jq
cp jq /usr/bin
jq --version

		info: Retrieved package metadata using apk default method. 

	File licenses found in Layer:  None
	Packages found in Layer: None  
 ```
 
 ***By orion***
 
```
##### Package: github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64

PackageName: github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
SPDXID: SPDXRef-Package-github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
PackageDownloadLocation: https://github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageFileName: github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION

FileName: jq
SPDXID: SPDXRef-File-d0be61a31640f44917ef03b5957d84137e7e1b77
FileChecksum: SHA1: 056ba5d6bbc617c29d37158ce11fd5a443093949
FileChecksum: SHA256: af986793a515d500ab2d35f8d2aecd656e764504b789b66d7e1a0b727a124c44
FileChecksum: SHA512: c9e585368bcb89d4c5213a31866e9301f03fe27165afcb4a3cdf0ec1be43b0fb7439d71dd9607ccc002622915b40389ee79c67d4c3c54ff95257cb23643b0330
LicenseConcluded: NOASSERTION
LicenseInfoInFile: NOASSERTION
FileCopyrightText: NOASSERTION

Relationship: SPDXRef-Package-github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 CONTAINS SPDXRef-File-d0be61a31640f44917ef03b5957d84137e7e1b77
FileName: usr/bin/jq
SPDXID: SPDXRef-File-98b8df3805ffe898dd72fdd564c864e66d5c9020
FileChecksum: SHA1: 056ba5d6bbc617c29d37158ce11fd5a443093949
FileChecksum: SHA256: af986793a515d500ab2d35f8d2aecd656e764504b789b66d7e1a0b727a124c44
FileChecksum: SHA512: c9e585368bcb89d4c5213a31866e9301f03fe27165afcb4a3cdf0ec1be43b0fb7439d71dd9607ccc002622915b40389ee79c67d4c3c54ff95257cb23643b0330
LicenseConcluded: NOASSERTION
LicenseInfoInFile: NOASSERTION
FileCopyrightText: NOASSERTION

Relationship: SPDXRef-Package-github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64 CONTAINS SPDXRef-File-98b8df3805ffe898dd72fdd564c864e66d5c9020

Relationship: SPDXRef-DOCUMENT-FOR-ADDONS DESCRIBES SPDXRef-Package-github.com/stedolan/jq/releases/download/jq-1.6/jq-linux64
```

## Scenario 3. Install packages that are managed by python package manager

### Comparison

1. **tern** unrecoganizes `wget` and reports it "Unrecognized Commands".
2. **tern** checks with the package manager on each image layer, recognizes and lists the installed packages by package manager.
3. **orion** recoganizes `wget` dowload operations  and treats each download as a spdx package with download location information. 
4. **orion** does not list individual python packages and lets it be handled by python package manager or other tools.

**Dockerfile segment**

```
RUN set -ex \
	&& apk add --no-cache --virtual .fetch-deps \
		gnupg \
		tar \
		xz \
	\
	&& wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" \
	&& wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" \
	&& export GNUPGHOME="$(mktemp -d)" \
	&& gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "$GPG_KEY" \
	&& gpg --batch --verify python.tar.xz.asc python.tar.xz \
	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } \
	&& rm -rf "$GNUPGHOME" python.tar.xz.asc \
	&& mkdir -p /usr/src/python \
	&& tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz \
	&& rm python.tar.xz \
	\
	&& apk add --no-cache --virtual .build-deps  \
		bluez-dev \
		bzip2-dev \
		coreutils \
		dpkg-dev dpkg \
		expat-dev \
		findutils \
		gcc \
		gdbm-dev \
		libc-dev \
		libffi-dev \
		libnsl-dev \
		libtirpc-dev \
		linux-headers \
		make \
		ncurses-dev \
		openssl-dev \
		pax-utils \
		readline-dev \
		sqlite-dev \
		tcl-dev \
		tk \
		tk-dev \
		util-linux-dev \
		xz-dev \
		zlib-dev \
# add build deps before removing fetch deps in case there's overlap
	&& apk del --no-network .fetch-deps \
	\
	&& cd /usr/src/python \
	&& gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" \
	&& ./configure \
		--build="$gnuArch" \
		--enable-loadable-sqlite-extensions \
		--enable-optimizations \
		--enable-option-checking=fatal \
		--enable-shared \
		--with-system-expat \
		--with-system-ffi \
		--without-ensurepip \
	&& make -j "$(nproc)" \
# set thread stack size to 1MB so we don't segfault before we hit sys.getrecursionlimit()
# https://github.com/alpinelinux/aports/commit/2026e1259422d4e0cf92391ca2d3844356c649d0
		EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000" \
		LDFLAGS="-Wl,--strip-all" \
	&& make install \
	&& rm -rf /usr/src/python \
	\
	&& find /usr/local -depth \
		\( \
			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) \
			-o \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name '*.a' \) \) \
			-o \( -type f -a -name 'wininst-*.exe' \) \
		\) -exec rm -rf '{}' + \
	\
	&& find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec scanelf --needed --nobanner --format '%n#p' '{}' ';' \
		| tr ',' '\n' \
		| sort -u \
		| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' \
		| xargs -rt apk add --no-cache --virtual .python-rundeps \
	&& apk del --no-network .build-deps \
	\
	&& python3 --version
 ```
 
**Outputs**

***By tern***

```
	Layer 3:
		info: Layer created by commands: /bin/sh -c set -ex 	&& apk add --no-cache --virtual .fetch-deps 		gnupg 		tar 		xz 		&& 
      wget -O python.tar.xz "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz" 	&& 
      wget -O python.tar.xz.asc "https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc" 	&& 
      export GNUPGHOME="$(mktemp -d)" 	&& gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys "$GPG_KEY" 	&& 
      gpg --batch --verify python.tar.xz.asc python.tar.xz 	&& { command -v gpgconf > /dev/null && gpgconf --kill all || :; } 	&& 
      rm -rf "$GNUPGHOME" python.tar.xz.asc 	&& mkdir -p /usr/src/python 	&& 
      tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz 	&& rm python.tar.xz 		&& 
      apk add --no-cache --virtual .build-deps  		bluez-dev 		bzip2-dev 		coreutils 		dpkg-dev dpkg 		expat-dev 		findutils 		gcc 		gdbm-dev 		libc-dev 		libffi-dev 		libnsl-dev 		libtirpc-dev 		linux-headers 		make 		ncurses-dev 		openssl-dev 		pax-utils 		readline-dev 		sqlite-dev 		tcl-dev 		tk 		tk-dev 		util-linux-dev 		xz-dev 		zlib-dev 	&& 
      apk del --no-network .fetch-deps 		&& cd /usr/src/python 	&& 
      gnuArch="$(dpkg-architecture --query DEB_BUILD_GNU_TYPE)" 	&& 
      ./configure 		--build="$gnuArch" 		--enable-loadable-sqlite-extensions 		--enable-optimizations 		--enable-option-checking=fatal 		--enable-shared 		--with-system-expat 		--with-system-ffi 		--without-ensurepip 	&& 
      make -j "$(nproc)" 		EXTRA_CFLAGS="-DTHREAD_STACK_SIZE=0x100000" 		LDFLAGS="-Wl,--strip-all" 	&& 
      make install 	&& rm -rf /usr/src/python 		
      && find /usr/local -depth 		\( 			\( -type d -a \( -name test -o -name tests -o -name idle_test \) \) 			-o \( -type f -a \( -name '*.pyc' -o -name '*.pyo' -o -name '*.a' \) \) 			-o \( -type f -a -name 'wininst-*.exe' \) 		\) -exec rm -rf '{}' + 		&& 
      find /usr/local -type f -executable -not \( -name '*tkinter*' \) -exec scanelf --needed --nobanner --format '%n#p' '{}' ';' 		| tr ',' '\n' 		| sort -u 		| awk 'system("[ -e /usr/local/lib/" $1 " ]") == 0 { next } { print "so:" $1 }' 		| xargs -rt apk add --no-cache --virtual .python-rundeps 	&& 
      apk del --no-network .build-deps 		&& python3 --version
		warning: 
Unrecognized Commands:set -ex
wget -O python.tar.xz https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz
wget -O python.tar.xz.asc https://www.python.org/ftp/python/${PYTHON_VERSION%%[a-z]*}/Python-$PYTHON_VERSION.tar.xz.asc
gpg --batch --keyserver hkps://keys.openpgp.org --recv-keys $GPG_KEY
gpg --batch --verify python.tar.xz.asc python.tar.xz
command -v gpgconf > /dev/null
gpgconf --kill all
rm -rf $GNUPGHOME python.tar.xz.asc
mkdir -p /usr/src/python
tar -xJC /usr/src/python --strip-components=1 -f python.tar.xz
rm python.tar.xz
cd /usr/src/python
--query DEB_BUILD_GNU_TYPE
./configure --build=$gnuArch --enable-loadable-sqlite-extensions --enable-optimizations --enable-option-checking=fatal --enable-shared --with-system-expat --with-system-ffi --without-ensurepip
make -j $(nproc) EXTRA_CFLAGS=-DTHREAD_STACK_SIZE=0x100000 LDFLAGS=-Wl,--strip-all
make install
rm -rf /usr/src/python
find /usr/local -depth ( ( -type d -a ( -name test -o -name tests -o -name idle_test ) ) -o ( -type f -a ( -name *.pyc -o -name *.pyo -o -name *.a ) ) -o ( -type f -a -name wininst-*.exe ) ) -exec rm -rf {} +
find /usr/local -type f -executable -not ( -name *tkinter* ) -exec scanelf --needed --nobanner --format %n#p
| tr , n | sort -u | awk system([ -e /usr/local/lib/ $1  ]) == 0 { next } { print so: $1 } | xargs -rt apk add --no-cache --virtual .python-rundeps
python3 --version

		info: Retrieved package metadata using apk default method. 

	File licenses found in Layer:  None
	Packages found in Layer: 
	+------------------------+------------------+---------+------------+
	| Package                | Version          | License | Pkg Format |
	+------------------------+------------------+---------+------------+
	| musl                   | 1.2.2-r3         |         | apk        |
	| busybox                | 1.33.1-r6        |         | apk        |
	| alpine-baselayout      | 3.2.0-r16        |         | apk        |
	| alpine-keys            | 2.4-r0           |         | apk        |
	| libcrypto1.1           | 1.1.1l-r0        |         | apk        |
	| libssl1.1              | 1.1.1l-r0        |         | apk        |
	| ca-certificates-bundle | 20191127-r5      |         | apk        |
	| libretls               | 3.3.3p1-r2       |         | apk        |
	| ssl_client             | 1.33.1-r6        |         | apk        |
	| zlib                   | 1.2.11-r3        |         | apk        |
	| apk-tools              | 2.12.7-r0        |         | apk        |
	| scanelf                | 1.3.2-r0         |         | apk        |
	| musl-utils             | 1.2.2-r3         |         | apk        |
	| libc-utils             | 0.7.2-r3         |         | apk        |
	| ca-certificates        | 20191127-r5      |         | apk        |
	| libffi                 | 3.3-r2           |         | apk        |
	| libintl                | 0.21-r0          |         | apk        |
	| ncurses-terminfo-base  | 6.2_p20210612-r0 |         | apk        |
	| ncurses-libs           | 6.2_p20210612-r0 |         | apk        |
	| libbz2                 | 1.0.8-r1         |         | apk        |
	| gdbm                   | 1.19-r0          |         | apk        |
	| sqlite-libs            | 3.35.5-r0        |         | apk        |
	| xz-libs                | 5.2.5-r0         |         | apk        |
	| expat                  | 2.4.1-r0         |         | apk        |
	| libtirpc-conf          | 1.3.2-r0         |         | apk        |
	| krb5-conf              | 1.0-r2           |         | apk        |
	| libcom_err             | 1.46.2-r0        |         | apk        |
	| keyutils-libs          | 1.6.3-r0         |         | apk        |
	| libverto               | 0.3.2-r0         |         | apk        |
	| krb5-libs              | 1.18.4-r0        |         | apk        |
	| libtirpc               | 1.3.2-r0         |         | apk        |
	| libnsl                 | 1.3.0-r0         |         | apk        |
	| libuuid                | 2.37.2-r0        |         | apk        |
	| readline               | 8.1.0-r0         |         | apk        |
	| .python-rundeps        | 20211213.165615  |         | apk        |
	+------------------------+------------------+---------+------------+
  ```

***By orion***

```
PackageName: www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz
SPDXID: SPDXRef-Package-www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz
PackageDownloadLocation: https://www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageFileName: www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION

Relationship: SPDXRef-DOCUMENT-FOR-ADDONS DESCRIBES SPDXRef-Package-www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.

PackageName: www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz.asc
SPDXID: SPDXRef-Package-www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz.asc
PackageDownloadLocation: https://www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz.asc
FilesAnalyzed: false
PackageLicenseConcluded: NOASSERTION
PackageFileName: www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz.asc
PackageLicenseDeclared: NOASSERTION
PackageCopyrightText: NOASSERTION

Relationship: SPDXRef-DOCUMENT-FOR-ADDONS DESCRIBES SPDXRef-Package-www.python.org/ftp/python/3.8.12/Python-3.8.12.tar.xz.asc
```

