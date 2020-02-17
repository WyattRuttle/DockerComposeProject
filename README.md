# DockerComposeProject
A simple Python web application running on Docker Compose. The application uses the Flask framework and maintains a hit counter in Redis
## Create a directory for the project
    mkdir dockerproject
    cd dockerproject
## Create the python file
    import time

    import redis
    from flask import Flask

    app = Flask(__name__)
    cache = redis.Redis(host='redis', port=6379)


    def get_hit_count():
        retries = 5
        while True:
            try:
                return cache.incr('hits')
            except redis.exceptions.ConnectionError as exc:
                if retries == 0:
                    raise exc
                retries -= 1
                time.sleep(0.5)


    @app.route('/')
    def hello():
        count = get_hit_count()
        return 'Hello World! I have been seen {} times.\n'.format(count)
## Create requirements file
    touch requirements.txt
    flask 
    redis
## Create Dockerfile
    FROM python:3.7-alpine
    WORKDIR /code
    ENV FLASK_APP app.py
    ENV FLASK_RUN_HOST 0.0.0.0
    RUN apk add --no-cache gcc musl-dev linux-headers
    COPY requirements.txt requirements.txt
    RUN pip install -r requirements.txt
    COPY . .
    CMD ["flask", "run"]
This is to build an image with the Python 3.7 image
Sets a working directory to /code
Sets environment variables used by the flask command
Copys the requirements.txt and installs the python dependencies

## Configure the Compose file
vi docker-compose.yml
    version: '3'
    services:
      web:
        build: .
        ports:
          - "5000:5000"
      redis:
        image: "redis:alpine"

## Set up the application
    docker-compose up

## Browse to IP:5000 in browser
Refresh the page and watch the counter go up 
