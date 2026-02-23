#---------STAGE 1: BUILD --------
FROM golang:1.22-alpine AS builder

WORKDIR /app

RUN apk add --no-cache git

COPY go.mod go.sum ./
RUN go mod download

COPY . .

# Build binary tu app.go
RUN CGO_ENABLED=0 GOOS=linux GOARCH=amd64 \
    go build -o server app.go


# -------- STAGE 2: RUNTIME --------
FROM alpine:3.19

WORKDIR /app

# Tao user non-root
RUN adduser -D appuser

# Copy binary
COPY --from=builder /app/server .

# Doi owner
RUN chown appuser:appuser /app/server

USER appuser

EXPOSE 8080

CMD ["./server"]
