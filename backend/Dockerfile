# =============================================
# Stage 1: Build
# =============================================
FROM eclipse-temurin:21-jdk-alpine AS builder

WORKDIR /app

# Cài dos2unix để xử lý Windows line endings
RUN apk add --no-cache dos2unix

# Copy Gradle wrapper và config trước để tận dụng layer cache
COPY gradlew settings.gradle build.gradle ./
COPY gradle ./gradle

# Fix Windows line endings (CRLF -> LF) và cấp quyền thực thi
RUN dos2unix gradlew && chmod +x gradlew

# Download dependencies (cache layer riêng)
RUN ./gradlew dependencies --no-daemon || true

# Copy source code
COPY src ./src

# Build JAR (bỏ qua test vì đã chạy local bằng Gradle)
RUN ./gradlew bootJar -x test --no-daemon

# =============================================
# Stage 2: Runtime
# =============================================
FROM eclipse-temurin:21-jre-alpine AS runtime

WORKDIR /app

# Tạo user non-root để tăng bảo mật
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Copy JAR từ stage build
COPY --from=builder /app/build/libs/campus-hub-1.0.0.jar app.jar

# Đổi quyền sở hữu
RUN chown appuser:appgroup app.jar

USER appuser

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
