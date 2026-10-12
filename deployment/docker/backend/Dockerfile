# ─── Stage 1: Build ───────────────────────────────────────────
FROM eclipse-temurin:17-jdk-alpine AS builder

WORKDIR /build

# Copy Maven wrapper & pom first (layer cache for deps)
COPY mvnw pom.xml ./
COPY .mvn .mvn
RUN ./mvnw dependency:go-offline -q

# Copy source & build fat JAR
COPY src ./src
RUN ./mvnw package -DskipTests -q

# Extract layered JAR for optimal layer caching
RUN java -Djarmode=layertools -jar target/*.jar extract --destination extracted

# ─── Stage 2: Runtime ─────────────────────────────────────────
FROM eclipse-temurin:17-jre-alpine AS runtime

# Security hardening: non-root user
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

WORKDIR /app

# Copy layered JAR content (best caching order: rarely → frequently changed)
COPY --from=builder --chown=appuser:appgroup /build/extracted/dependencies ./
COPY --from=builder --chown=appuser:appgroup /build/extracted/spring-boot-loader ./
COPY --from=builder --chown=appuser:appgroup /build/extracted/snapshot-dependencies ./
COPY --from=builder --chown=appuser:appgroup /build/extracted/application ./

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "org.springframework.boot.loader.launch.JarLauncher"]