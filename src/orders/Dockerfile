
# -------- Build Stage --------
FROM ubuntu:24.04 as build-env

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    maven \
    openjdk-21-jdk-headless \
    curl \
    tar \
    gzip \
    ca-certificates \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /

# Copy Maven wrapper & config
COPY .mvn .mvn
COPY mvnw .
COPY pom.xml .

RUN chmod +x mvnw && ./mvnw dependency:go-offline -B -q

# Copy source
COPY ./src ./src

# Build JAR
RUN ./mvnw -DskipTests package -q && \
    mv /target/orders-0.0.1-SNAPSHOT.jar /app.jar


# -------- Runtime Stage --------
FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    openjdk-21-jre-headless \
    curl \
    && rm -rf /var/lib/apt/lists/*

# Create non-root user
ENV APPUSER=appuser
ENV APPUID=1000
ENV APPGID=1000

RUN groupadd -g $APPGID $APPUSER && \
    useradd -m -u $APPUID -g $APPGID $APPUSER

ENV JAVA_TOOL_OPTIONS=
ENV SPRING_PROFILES_ACTIVE=prod

WORKDIR /app

# Copy files
COPY ./ATTRIBUTION.md ./LICENSES.md
COPY --from=build-env /app.jar .

RUN chown -R $APPUSER:$APPUSER /app

USER $APPUSER

EXPOSE 8080

ENTRYPOINT ["sh", "-c", "java $JAVA_OPTS -jar /app/app.jar"]