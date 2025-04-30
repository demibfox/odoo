FROM ubuntu:noble

SHELL ["/bin/bash", "-xo", "pipefail", "-c"]

# Generate locale C.UTF-8 for postgres and general locale data
ENV LANG=en_US.UTF-8

# Install system dependencies
RUN apt-get update && \
    DEBIAN_FRONTEND=noninteractive \
    apt-get install -y --no-install-recommends \
        build-essential \
        ca-certificates \
        curl \
        dirmngr \
        fonts-noto-cjk \
        gcc \
        git \
        gnupg \
        libldap2-dev \
        libpq-dev \
        libsasl2-dev \
        libssl-dev \
        node-less \
        npm \
        python3-dev \
        python3-magic \
        python3-num2words \
        python3-odf \
        python3-pdfminer \
        python3-pip \
        python3-phonenumbers \
        python3-pyldap \
        python3-qrcode \
        python3-renderpm \
        python3-setuptools \
        python3-slugify \
        python3-venv \
        python3-vobject \
        python3-watchdog \
        python3-wheel \
        python3-xlrd \
        python3-xlwt \
        xz-utils && \
    rm -rf /var/lib/apt/lists/*

# Install wkhtmltopdf
RUN curl -o wkhtmltox.deb -sSL https://github.com/wkhtmltopdf/packaging/releases/download/0.12.6.1-3/wkhtmltox_0.12.6.1-3.jammy_amd64.deb \
    && apt-get update \
    && apt-get install -y --no-install-recommends ./wkhtmltox.deb \
    && rm -rf /var/lib/apt/lists/* wkhtmltox.deb

# Install PostgreSQL client
RUN echo 'deb http://apt.postgresql.org/pub/repos/apt/ noble-pgdg main' > /etc/apt/sources.list.d/pgdg.list \
    && curl -fsSL https://www.postgresql.org/media/keys/ACCC4CF8.asc | gpg --dearmor -o /etc/apt/trusted.gpg.d/postgresql.gpg \
    && apt-get update \
    && apt-get install -y postgresql-client \
    && rm -rf /var/lib/apt/lists/*

# Install rtlcss
RUN npm install -g rtlcss

# Create odoo user and directories
RUN useradd -m -U -r -s /bin/bash odoo && \
    mkdir -p /etc/odoo /mnt/extra-addons /var/lib/odoo /opt/odoo && \
    chown -R odoo:odoo /etc/odoo /mnt/extra-addons /var/lib/odoo /opt/odoo

# Set up environment
ENV ODOO_VERSION=18.0
ENV VIRTUAL_ENV=/opt/odoo/venv
ENV PATH="$VIRTUAL_ENV/bin:$PATH"
ENV ODOO_RC=/etc/odoo/odoo.conf

# Switch to odoo user for git operations
USER odoo
WORKDIR /tmp

# Clone Odoo source
RUN git clone --depth=1 -b ${ODOO_VERSION} https://github.com/demibfox/odoo.git && \
    mv odoo/* odoo/.git* /opt/odoo/ && \
    cd /opt/odoo && \
    python3 -m venv $VIRTUAL_ENV && \
    pip install --upgrade pip wheel && \
    pip install --no-cache-dir -r requirements.txt

WORKDIR /opt/odoo

# Set up configuration and scripts
RUN cp docker/odoo.conf /etc/odoo/ && \
    cp docker/entrypoint.sh /usr/local/bin/ && \
    cp docker/wait-for-psql.py /usr/local/bin/

# Switch back to root to set permissions
USER root
RUN chmod +x /usr/local/bin/entrypoint.sh /usr/local/bin/wait-for-psql.py

# Switch back to odoo user for running the service
USER odoo

VOLUME ["/var/lib/odoo", "/mnt/extra-addons"]
EXPOSE 8069 8071 8072

ENTRYPOINT ["entrypoint.sh"]
CMD ["./odoo-bin"]
