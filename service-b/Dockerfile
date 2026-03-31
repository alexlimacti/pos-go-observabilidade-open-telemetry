# Stage 1: Build the Go application
FROM golang:1.22-alpine AS builder

WORKDIR /app

COPY go.mod ./
RUN go mod download

COPY *.go ./

RUN CGO_ENABLED=0 GOOS=linux go build -o /service-b

# Stage 2: Create the final lightweight image
FROM alpine:latest

WORKDIR /

COPY --from=builder /service-b /service-b

EXPOSE 8081

CMD ["/service-b"]
