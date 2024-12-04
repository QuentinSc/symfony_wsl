FROM php:8.2-fpm

# Install system dependencies
RUN apt-get update && apt-get install -y \
    git \
    curl \
    libpng-dev \
    libonig-dev \
    libxml2-dev \
    zip \
    unzip

# Clear cache
RUN apt-get clean && rm -rf /var/lib/apt/lists/*

# Install PHP extensions
RUN docker-php-ext-install pdo_mysql mbstring exif pcntl bcmath gd

# Get latest Composer
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer

# Configure permissions
RUN mkdir -p /var/www/html && \
    mkdir -p /var/www/.composer && \
    chown -R www-data:www-data /var/www/html && \
    chown -R www-data:www-data /var/www/.composer && \
    chmod 777 /var/www/html && \
    chmod 777 /var/www/.composer

# Set working directory
WORKDIR /var/www/html

USER www-data