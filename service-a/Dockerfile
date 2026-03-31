# Stage 1: Build the Go application
FROM golang:1.22-alpine AS builder

WORKDIR /app

COPY go.mod ./
RUN go mod download

COPY *.go ./

RUN CGO_ENABLED=0 GOOS=linux go build -o /service-a

# Stage 2: Create the final lightweight image
FROM alpine:latest

WORKDIR /

COPY --from=builder /service-a /service-a

EXPOSE 8080

CMD ["/service-a"]
