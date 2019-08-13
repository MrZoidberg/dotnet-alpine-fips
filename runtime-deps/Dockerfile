FROM mikhailmerkulov/alpine-fips:3.9

LABEL maintainer="Mikhail Merkulov <mihail.merkulov@gmail.com>"

RUN apk add --no-cache \
        ca-certificates \
        \
        # .NET Core dependencies
        krb5-libs \
        libgcc \
        libintl \
#        libssl1.0 \
        libstdc++ \
        lttng-ust \
        tzdata \
        userspace-rcu \
        zlib \        
    && rm -rf /var/cache/apk/*

# Configure web servers to bind to port 80 when present
ENV ASPNETCORE_URLS=http://+:80 \
    # Enable detection of running in a container
    DOTNET_RUNNING_IN_CONTAINER=true \
    # Set the invariant mode since icu_libs isn't included (see https://github.com/dotnet/announcements/issues/20)
    DOTNET_SYSTEM_GLOBALIZATION_INVARIANT=true

# add AWS RDS CA bundle
ADD https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem /tmp/rds-ca/aws-rds-ca-bundle.pem
# split the bundle into individual certs (prefixed with xx)
# see http://blog.swwomm.com/2015/02/importing-new-rds-ca-certificate-into.html
RUN cd /tmp/rds-ca && cat aws-rds-ca-bundle.pem|awk 'split_after==1{n++;split_after=0} /-----END CERTIFICATE-----/ {split_after=1} {print > "cert" n ""}' \
    && for CERT in /tmp/rds-ca/cert*; do mv $CERT /usr/local/share/ca-certificates/aws-rds-ca-$(basename $CERT).crt; done \
    && rm -rf /tmp/rds-ca \
    && update-ca-certificates