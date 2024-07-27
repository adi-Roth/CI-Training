# My Calculator Project

This is a simple Python project that demonstrates the following:
- A basic calculator module with add, subtract, multiply, and divide functions.
- Unit tests for the calculator functions.
- A multi-stage Dockerfile for building and deploying the application.

## Project Structure

├── app/
│ ├── init.py
│ ├── calculator.py
├── tests/
│ ├── init.py
│ ├── test_calculator.py
├── Dockerfile
├── requirements.txt
└── README.md

## How to Run

### Using Docker

1. Build the Docker image:
    ```sh
    docker build -t my_calculator_project .
    ```

2. Run the Docker container:
    ```sh
    docker run -it my_calculator_project
    ```

### Running Tests

1. Install dependencies:
    ```sh
    pip install -r requirements.txt
    ```

2. Run the tests:
    ```sh
    python -m unittest discover -s tests
    ```

## License

This project is licensed under the MIT License.
