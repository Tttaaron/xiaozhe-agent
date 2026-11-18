FROM node:22-bookworm-slim AS frontend-builder
WORKDIR /workspace/frontend

COPY frontend/package*.json ./
# 课程学员的 Docker 环境差异较大；前端依赖安装优先选择 glibc 基础镜像并避免 npm cache mount，
# 降低 Alpine/musl + BuildKit 缓存触发 npm/cli Exit Handler 竞态的概率。
RUN npm ci --no-audit --no-fund --loglevel=warn

COPY frontend ./
RUN npm run build

FROM maven:3.9.6-eclipse-temurin-17 AS builder
WORKDIR /workspace

COPY docker ./docker
RUN mkdir -p /root/.m2 \
    && cp docker/maven-settings.xml /root/.m2/settings.xml

COPY pom.xml .

# 兼容经典构建器（无 BuildKit）：不用 --mount=type=cache
RUN mkdir -p /root/.m2/repository \
    && if [ -f docker/m2-cache.tar.gz ]; then tar -xzf docker/m2-cache.tar.gz -C /root/.m2/repository; fi \
    && find /root/.m2/repository \( -name '*.lastUpdated' -o -name 'resolver-status.properties' \) -delete \
    && if [ -f docker/m2-cache.tar.gz ]; then \
        mvn -o -s /root/.m2/settings.xml -q -DskipTests dependency:go-offline \
        || mvn -s /root/.m2/settings.xml -q -DskipTests dependency:go-offline; \
    else \
        mvn -s /root/.m2/settings.xml -q -DskipTests dependency:go-offline; \
    fi

COPY src ./src
COPY --from=frontend-builder /workspace/frontend/dist ./frontend/dist
RUN mvn -s /root/.m2/settings.xml -q -DskipTests package

FROM eclipse-temurin:17-jre
WORKDIR /app
COPY --from=builder /workspace/target/ecommerce-backend-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8081
ENTRYPOINT ["java", "-jar", "app.jar"]
