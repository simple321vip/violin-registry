# 第一阶段：构建环境
FROM postgres:16.3-alpine AS builder

# 删除非必要文件（文档、调试符号、静态库等）
RUN apk add --no-cache findutils binutils && \
  find / -name '*.doc' -delete && \
  find / -name '*.a' -delete && \
  find / -name '*.la' -delete && \
  find /usr -name 'test' -exec rm -rf {} + && \
  strip /usr/local/bin/postgres && \
  rm -rf /var/cache/apk/*

# 第二阶段：生成最终镜像
FROM alpine:3.19

# 从构建阶段复制必要文件
COPY --from=builder / /

# 设置运行时依赖和用户
RUN apk add --no-cache libpq tzdata su-exec && \
  addgroup -S postgres && \
  adduser -S postgres -G postgres && \
  mkdir -p /var/lib/postgresql/data && \
  chown -R postgres:postgres /var/lib/postgresql/data

# 环境变量
ENV POSTGRES_USER=postgres \
  POSTGRES_PASSWORD=postgres \
  PGDATA=/var/lib/postgresql/data

# 数据卷和健康检查
VOLUME /var/lib/postgresql/data
HEALTHCHECK --interval=30s --timeout=5s \
  CMD pg_isready -U $POSTGRES_USER -d postgres

# 启动命令
USER postgres
EXPOSE 5432
CMD ["postgres", "-c", "config_file=/etc/postgresql/postgresql.conf"]