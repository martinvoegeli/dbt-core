ARG py_version=3.11.2

FROM python:$py_version-slim-bullseye as base

# Set environment variables
ENV VIRTUAL_ENV=/opt/venv
ENV PATH="$VIRTUAL_ENV/bin:/opt/mssql-tools18/bin:$PATH"


RUN apt-get update \
  && apt-get dist-upgrade -y \
  && apt-get install -y --no-install-recommends \
  build-essential=12.9 \
  ca-certificates=20210119 \
  git=1:2.30.2-1+deb11u2 \
  libpq-dev=13.14-0+deb11u1 \
  make=4.3-4.1 \
  openssh-client=1:8.4p1-5+deb11u3 \
  software-properties-common=0.96.20.2-2.1 \
  curl \
  && apt-get clean \
  && rm -rf \
  /var/lib/apt/lists/* \
  /tmp/* \
  /var/tmp/*

# Set up the Python virtual environment
RUN python3 -m venv $VIRTUAL_ENV

ENV PYTHONIOENCODING=utf-8
ENV LANG=C.UTF-8

RUN python -m pip install --upgrade "pip==24.0" "setuptools==69.2.0" "wheel==0.43.0" --no-cache-dir


FROM base as dbt-core

ARG commit_ref=main

HEALTHCHECK CMD dbt --version || exit 1

WORKDIR /usr/app/dbt/
ENTRYPOINT ["dbt"]

RUN python -m pip install --no-cache-dir "dbt-core @ git+https://github.com/dbt-labs/dbt-core@${commit_ref}#subdirectory=core"




#Install the odbc driver as a precondition
RUN curl https://packages.microsoft.com/keys/microsoft.asc | tee /etc/apt/trusted.gpg.d/microsoft.asc

#Download appropriate package for the OS version
#Choose only ONE of the following, corresponding to your OS version


#Debian 11
RUN curl https://packages.microsoft.com/config/debian/11/prod.list | tee /etc/apt/sources.list.d/mssql-release.list

RUN apt-get update
RUN ACCEPT_EULA=Y apt-get install -y msodbcsql18
# optional: for bcp and sqlcmd
RUN ACCEPT_EULA=Y apt-get install -y mssql-tools18

# optional: for unixODBC development headers
RUN apt-get install -y unixodbc-dev
# optional: kerberos library for debian-slim distributions
RUN apt-get install -y libgssapi-krb5-2


RUN pip install -U dbt-fabric


FROM base as dbt-postgres

ARG commit_ref=main

HEALTHCHECK CMD dbt --version || exit 1

WORKDIR /usr/app/dbt/
ENTRYPOINT ["dbt"]

RUN python -m pip install --no-cache-dir "dbt-postgres @ git+https://github.com/dbt-labs/dbt-core@${commit_ref}#subdirectory=plugins/postgres"


FROM dbt-core as dbt-third-party

ARG dbt_third_party

RUN if [ "$dbt_third_party" ]; then \
  python -m pip install --no-cache-dir "${dbt_third_party}"; \
  else \
  echo "No third party adapter provided"; \
  fi \
