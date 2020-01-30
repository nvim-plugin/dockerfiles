# syntax = docker/dockerfile:1.1.3-experimental

# target=buster-slim
FROM docker.io/debian:buster-slim AS buster-slim
ARG LLVM_VERSION
RUN set -eux && \
	apt-get update && \
	apt-get install -y --no-install-recommends \
		# buildpack-deps/buster-curl
		ca-certificates \
		curl \
		netbase \
		wget \
		gnupg \
		dirmngr \
		&& \
		touch /etc/apt/sources.list.d/llvm.list && \
		echo "deb http://apt.llvm.org/buster/ llvm-toolchain-buster-${LLVM_VERSION} main" >> /etc/apt/sources.list.d/llvm.list && \
		echo "deb-src http://apt.llvm.org/buster/ llvm-toolchain-buster-${LLVM_VERSION} main" >> /etc/apt/sources.list.d/llvm.list && \
		\
		wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | apt-key add - && \
		rm -f /etc/apt/apt.conf.d/docker-clean; echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt \
	set -eux && \
	apt-get update && \
	apt-get install -y --no-install-recommends \
		# buildpack-deps/buster-scm
		git \
		mercurial \
		openssh-client \
		procps \
		subversion \
		\
		# buildpack-deps/buster
		autoconf \
		automake \
		cmake \
		g++ \
		gcc \
		gettext \
		libc6-dev \
		libtool \
		libtool-bin \
		make \
		ninja-build \
		pkg-config \
		unzip \
		\
		# llvm
		libllvm${LLVM_VERSION} llvm-${LLVM_VERSION} llvm-${LLVM_VERSION}-dev llvm-${LLVM_VERSION}-runtime \
		clang-${LLVM_VERSION} clang-tools-${LLVM_VERSION} libclang-common-${LLVM_VERSION}-dev libclang-${LLVM_VERSION}-dev libclang1-${LLVM_VERSION} \
		libc++-${LLVM_VERSION}-dev libc++abi-${LLVM_VERSION}-dev

# target=nvim-builder
FROM buster-slim AS nvim-builder
RUN set -ex && \
	git clone --depth 1 --branch master --single-branch https://github.com/neovim/neovim.git /src/neovim
RUN set -ex && \
	CC="clang-${LLVM_VERSION}" CXX="clang++-${LLVM_VERSION}" LD="/usr/bin/lld-${LLVM_VERSION}" CFLAGS='-march=native' VERBOSE=1 \
		make -C /src/neovim install \
			CMAKE_BUILD_TYPE=Release \
			CMAKE_INSTALL_PREFIX=/out/usr/local \
			DEPS_CMAKE_FLAGS="-DCMAKE_C_COMPILER=clang-${LLVM_VERSION} -DCMAKE_LINKER=/usr/bin/lld-${LLVM_VERSION}" \
			CMAKE_EXTRA_FLAGS="-DCMAKE_C_COMPILER=clang-${LLVM_VERSION} -DCMAKE_LINKER=/usr/bin/lld-${LLVM_VERSION} -DMIN_LOG_LEVEL=0"

# target=nvim
FROM docker.io/golang:1.13-buster AS nvim
LABEL org.label-schema.schema-version = "1.0"
LABEL org.label-schema.name = "quay.io/neovim/go"
LABEL org.label-schema.description = "go container image with nvim binayr for testing"
LABEL org.label-schema.url="https://github.com/nvim-plugin/dockerfiles"
LABEL org.label-schema.vcs-url = "https://github.com/nvim-plugin/dockerfiles/blob/master/go/Dockerfile"
COPY --from=nvim-builder /out/ /
