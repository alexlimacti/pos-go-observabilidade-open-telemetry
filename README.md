# Go Observability with OpenTelemetry

This project demonstrates a distributed system built with Go, consisting of two microservices that work together to provide weather information for a given Brazilian ZIP code (CEP). The key feature of this project is the implementation of observability using OpenTelemetry (OTEL) for distributed tracing, with Zipkin as the backend for trace visualization.

## System Architecture

The system is composed of the following components:

-   **Service A (Input Service)**: The entry point for user requests. It receives a ZIP code, validates it, and forwards the request to Service B.
-   **Service B (Orchestration Service)**: Receives a valid ZIP code, fetches the corresponding city from the [ViaCEP API](https://viacep.com.br/), retrieves the weather for that city using the [WeatherAPI](https://www.weatherapi.com/), and returns the temperature in Celsius, Fahrenheit, and Kelvin.
-   **OpenTelemetry Collector**: Receives trace data from both services and exports it to Zipkin.
-   **Zipkin**: A distributed tracing system that allows for the visualization of the entire request flow.

All services are containerized and managed using Docker Compose.

## Prerequisites

-   [Docker](https://www.docker.com/get-started) and [Docker Compose](https://docs.docker.com/compose/install/) installed on your machine.
-   A free API key from [WeatherAPI](https://www.weatherapi.com/signup.aspx).

## How to Run

1.  **Clone the repository:**
    ```bash
    git clone <repository-url>
    cd <repository-folder>
    ```

2.  **Configure the WeatherAPI Key:**
    Open the `docker-compose.yaml` file and replace the placeholder `<YOUR_API_KEY>` with your actual WeatherAPI key:
    ```yaml
    # In service-b environment variables
    environment:
      - WEATHER_API_KEY=<YOUR_API_KEY>
      - OTEL_EXPORTER_OTLP_ENDPOINT=otel-collector:4318
    ```
    > **Note:** If you don't provide a valid API key, Service B will return a mocked temperature of 25°C for demonstration purposes.

3.  **Start the services:**
    Run the following command to build and start all the containers in detached mode:
    ```bash
    docker-compose up --build -d
    ```
    This will start Service A, Service B, the OTEL Collector, and Zipkin.

## How to Use

### Make a Request

To get the weather information, send a `POST` request to `Service A` on port `8080`.

Here is an example using `curl`:

```bash
curl --location --request POST 'http://localhost:8080' 
--header 'Content-Type: application/json' 
--data-raw '{
    "cep": "29902555"
}'
```

**Possible Responses:**

-   **Success (200 OK):**
    ```json
    {
        "city": "Linhares",
        "temp_C": 25.0,
        "temp_F": 77,
        "temp_K": 298
    }
    ```
-   **Invalid ZIP Code (422 Unprocessable Entity):**
    ```
    invalid zipcode
    ```
-   **ZIP Code Not Found (404 Not Found):**
    ```
    can not find zipcode
    ```

### View Traces in Zipkin

After making one or more requests, you can visualize the distributed traces in Zipkin.

1.  Open your web browser and navigate to the Zipkin UI:
    [http://localhost:9411](http://localhost:9411)

2.  Click the "Run Query" button to find recent traces. You should see traces that correspond to your API requests.

3.  Click on a specific trace to see the full waterfall diagram, showing the time spent in each service and for each external API call. You will see the complete flow from `service-a` to `service-b` and the custom spans for the `ViaCEP` and `WeatherAPI` calls.
