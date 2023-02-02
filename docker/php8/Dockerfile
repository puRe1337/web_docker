#
#
# Multistage Ghostscript
# Multistage is used to not compile Ghostscript over and over again everytime something is changed
# inside the Dockerfile.
#
#
FROM debian:11-slim as ghostscript

WORKDIR /gs

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    ca-certificates \
    libxext-dev && \
    curl -L \
    https://github.com/ArtifexSoftware/ghostpdl-downloads/releases/download/gs1000/ghostscript-10.0.0.tar.gz \
    | tar -xz \
    && cd ghostscript-10.0.0 && ./configure --prefix=/gs && make install -j$(nproc) \
    && cd .. && rm -rf ghostscript-10.0.0 ghostscript-10.0.0.tar.gz \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

#
#
# Multistage ImageMagick
# Multistage is used to not compile ImageMagick over and over again everytime something is changed
# inside the Dockerfile.
#
#
FROM debian:11-slim as imagemagick

WORKDIR /im
ENV LD_LIBRARY_PATH=/usr/local/lib
# Dependencies https://askubuntu.com/a/1357582

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
    build-essential \
    curl \
    git \
    ca-certificates \
    chrpath debhelper dh-exec dpkg-dev g++ ghostscript gsfonts libbz2-dev \
    libdjvulibre-dev libexif-dev libfftw3-dev libfontconfig1-dev libfreetype6-dev \
    libjpeg-dev liblcms2-dev liblqr-1-0-dev libltdl-dev liblzma-dev libopenexr-dev \
    libpango1.0-dev libperl-dev libpng-dev librsvg2-bin librsvg2-dev libtiff-dev libwebp-dev \
    libwmf-dev libx11-dev libxext-dev libxml2-dev libxt-dev pkg-config pkg-kde-tools zlib1g-dev \
    autoconf automake pkg-config \
    libde265-dev libopenjp2-7-dev libtool x265 \
    && git clone https://github.com/strukturag/libheif.git libheif \
    && cd libheif && ./autogen.sh && ./configure && make -j$(nproc) && make install -j$(nproc) \
    && cd .. && rm -rf libheif \
    && git clone https://github.com/ImageMagick/ImageMagick.git ImageMagick \
    && cd ImageMagick && git checkout tags/7.1.0-58 && ./configure && make -j$(nproc) && make install -j$(nproc) \
    && cd .. && rm -rf ImageMagick \
    && apt-get clean && rm -rf /var/lib/apt/lists/*
#
#
# Main php configuration
#
#

FROM php:8.0-fpm

# Arguments defined in docker-compose.yml
ARG user
ARG uid

# Install system dependencies
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    git \
    curl \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libjpeg62-turbo-dev \
    libmcrypt-dev \
    libgd-dev \
    jpegoptim optipng pngquant gifsicle \
    libonig-dev \
    libxml2-dev \
    libssl-dev \
    openssl \
    zip \
    unzip \
    lsb-release \
    libxext-dev \
    autoconf pkg-config libtool-bin \
    && rm -rf /var/lib/apt/lists/*
    # ghostscript # -> Old version

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Copy ghostscript from multistage
COPY --from=ghostscript /gs/. /usr/local

# Copy ImageMagick from multistage
COPY --from=imagemagick /usr/local/ /usr/local/
ENV LD_LIBRARY_PATH=/usr/local/lib
RUN ldconfig /usr/local/lib && ldconfig

# Install PHP extensions installer
COPY --from=mlocati/php-extension-installer:2.0.2 /usr/bin/install-php-extensions /usr/local/bin/

# Install PHP extensions
RUN install-php-extensions bcmath \
    bz2 \
    calendar \
    Core \
    ctype \
    curl \
    date \
    dba \
    dom \
    exif \
    FFI \
    fileinfo \
    filter \
    ftp \
    gd \
    gettext \
    gmp \
    hash \
    iconv \
    intl \
    json \
    ldap \
    libxml \
    mbstring \
    mysqli \
    mysqlnd \
    odbc \
    openssl \
    opcache \
    pcntl \
    pcre \
    PDO \
    pdo_dblib \
    pdo_mysql \
    PDO_ODBC \
    pdo_pgsql \
    pdo_sqlite \
    pgsql \
    Phar \
    posix \
    pspell \
    redis \
    readline \
    Reflection \
    session \
    shmop \
    SimpleXML \
    soap \
    sockets \
    sodium \
    SPL \
    sqlite3 \
    standard \
    sysvmsg \
    sysvsem \
    sysvshm \
    tidy \
    tokenizer \
    xml \
    xmlreader \
    xmlwriter \
    xsl \
    zip \
    zlib \
    imagick

# Clear cache again
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Get Composer
COPY --from=composer:2.4.4 /usr/bin/composer /usr/bin/composer

# Get FFMpeg
COPY --from=mwader/static-ffmpeg:5.1.2 /ffmpeg /usr/local/bin/
COPY --from=mwader/static-ffmpeg:5.1.2 /ffprobe /usr/local/bin/

# Create system user to run Composer and Artisan Commands
RUN useradd -G www-data,root -u $uid -d /home/$user $user
RUN mkdir -p /home/$user/.composer && \
    chown -R $user:$user /home/$user

# Install Node.js and NPM
COPY --from=node:18.13.0 /usr/local/ /usr/local/
RUN nodejs --version && node --version && npm --version

# opcache config
ENV PHP_OPCACHE_VALIDATE_TIMESTAMPS="0"
ADD ./docker/php8/opcache.ini "$PHP_INI_DIR/conf.d/opcache.ini"

# Set PHP Memory Limit to 512MB
ADD ./docker/php8/memory_limit.ini "$PHP_INI_DIR/conf.d/docker-php-memlimit.ini"

# Copy development php configuration to php.ini
RUN cp /usr/local/etc/php/php.ini-production /usr/local/etc/php/php.ini

# Set working directory
WORKDIR /var/www

USER $user
