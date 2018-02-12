# arma3.github.io

## Setting up the Jekyll environment

### Using Docker

We include files for [Docker](https://www.docker.com/) to run Jekyll in a separate container. This allows you to not having to install anything apart from Docker on your computer.

#### Running the Dockerfile

- Install [Docker](https://www.docker.com/)
- Open a CLI e.g. Powershell and navigate to the repo directory
    ```
    cd <arma3.github.io>
    ```

- Build and run the container
    ```
    docker-compose up
    ```

- Navigate to [http://localhost:4000](http://localhost:4000)
