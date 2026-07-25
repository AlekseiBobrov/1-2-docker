# Stage 1: Build the Go binary
FROM golang:1.17-alpine AS builder

RUN apk add --no-cache git ca-certificates

WORKDIR /src

COPY go.mod go.sum ./
RUN go mod download

COPY . .

RUN CGO_ENABLED=0 GOOS=linux go build -o /app ./cmd/api/

# Stage 2: Build runtime image
FROM alpine:3.15

RUN apk add --no-cache ca-certificates tzdata wget

RUN adduser -D -u 1001 appuser

COPY --from=builder /app /app

USER appuser

EXPOSE 8081

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
    CMD wget --no-verbose --tries=1 -qO- http://localhost:8081/health || exit 1

CMD ["/app"]