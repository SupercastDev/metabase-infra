# syntax=docker/dockerfile:1

# Metabase on a JDK we pin ourselves.
#
# The official metabase/metabase image bundles its own Temurin JRE, so JDK
# security fixes wait on Metabase shipping a new image. This takes the jar,
# certs and entrypoint script from the official image (pinned by digest) and
# runs them on a JRE we choose.
#
# Bump either ARG (tag AND digest) to upgrade Metabase or the JDK independently.

# Metabase v0.65.5.1 (2026-11-17)
ARG METABASE_IMAGE=metabase/metabase:v0.65.5.1@sha256:33b19ba4d6588c29051f2ee62f9166d54cb8beccb0d5513acaffc7bedae9aa71
# Corretto 25.0.5.1.2 - fixes CVE-2026-114938 (Java4Shell). Temurin 25 had no patched build yet.
ARG JAVA_IMAGE=amazoncorretto:25-alpine3.22@sha256:d693ce72ed3fae19ca83ce73453ed553d634919d536604c642fa5f6336534995

FROM ${METABASE_IMAGE} AS metabase

FROM ${JAVA_IMAGE}

ENV FC_LANG=en-US LC_CTYPE=en_US.UTF-8

# Same packages as the official image: bash for run_metabase.sh, fonts for
# PDF exports and static visualisations in dashboard subscriptions.
RUN apk add --no-cache bash fontconfig font-noto font-noto-arabic font-noto-hebrew font-noto-cjk \
 && apk upgrade --no-cache \
 && mkdir -p /opt/java \
 && ln -s "$JAVA_HOME" /opt/java/openjdk

COPY --from=metabase /app /app

# The official image trusts the RDS and DigiCert G2 roots for warehouse
# connections; import them into Corretto's truststore too.
RUN keytool -noprompt -importcert -trustcacerts -cacerts -storepass changeit \
      -alias aws-rds -file /app/certs/rds-combined-ca-bundle.pem \
 && keytool -noprompt -importcert -trustcacerts -cacerts -storepass changeit \
      -alias azure-cert -file /app/certs/DigiCertGlobalRootG2.crt.pem \
 && mkdir -p /plugins \
 && chmod a+rwx /plugins

EXPOSE 3000

ENTRYPOINT ["/app/run_metabase.sh"]
