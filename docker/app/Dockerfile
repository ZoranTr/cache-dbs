FROM php:7.4-fpm

# add composer for alpine linux
RUN curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

RUN apt-get update && apt-get install -y \
    build-essential \
    autoconf \
    automake \
    libpcre3-dev \
    libevent-dev \
    pkg-config \
    zlib1g-dev \
    libssl-dev \
    git

RUN git clone https://github.com/RedisLabs/memtier_benchmark.git && \
    cd memtier_benchmark && \
    autoreconf -ivf && \
    ./configure && \
    make && \
    make install
