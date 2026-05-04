# rizin docker for Mayhem fuzzing

FROM debian:12

# rz-pipe python version
ARG RZ_PIPE_PY_VERSION=master

ARG with_arm32_as
ARG with_arm64_as
ARG with_ppc_as

ENV RZ_PIPE_PY_VERSION ${RZ_PIPE_PY_VERSION}

# Build rizin in a volume to minimize space used by build
COPY . /tmp/rizin/

RUN apt-get update
RUN apt-get install -y --no-install-recommends \
	cmake \
	gcc \
	cpp \
	g++ \
	git \
	make \
	pkg-config \
	libc-dev-bin libc6-dev linux-libc-dev \
	python3-pip \
	python3-setuptools \
	python3-wheel \
	clang-13

RUN pip3 install --break-system-packages -I meson==0.61.5 ninja==1.10.2.4

WORKDIR /tmp/rizin
RUN meson --prefix=/usr /tmp/build
RUN meson compile -C /tmp/build
RUN meson install --destdir /tmp/rizin-install -C /tmp/build

ENV PKG_CONFIG_PATH=/tmp/rizin-install/usr/lib/pkgconfig/
ENV LD_LIBRARY_PATH=/tmp/rizin-install/usr/lib/

RUN clang++-13 /tmp/rizin/mayhem/fuzz_rz_asm_token_string_new.cpp \
    -fsanitize=fuzzer,address \
    -I /tmp/rizin-install/usr/include/librz/ \
    -I /tmp/rizin-install/usr/include/librz/sdb/ \
    -L /tmp/rizin-install/usr/lib/ \
    -Wl,--start-group \
    -lrz_asm -lrz_util -lrz_crypto -lrz_io -lrz_flag -lrz_reg -lrz_type \
    -Wl,--end-group \
    -o /fuzz

FROM debian:12

RUN apt-get update && apt-get install -y --no-install-recommends ca-certificates git python3

WORKDIR /home/rizin
ENV HOME /home/rizin

COPY --from=0 /tmp/rizin-install/ /
COPY --from=0 /fuzz /fuzz_rz_asm_token_string_new
