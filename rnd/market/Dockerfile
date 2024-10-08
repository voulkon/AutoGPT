FROM python:3.11-slim-buster AS builder

# Set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

WORKDIR /app

# Install build dependencies
RUN apt-get update \
    && apt-get install -y build-essential curl ffmpeg wget libcurl4-gnutls-dev libexpat1-dev gettext libz-dev libssl-dev postgresql-client git \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

ENV POETRY_VERSION=1.8.3 \
    POETRY_HOME="/opt/poetry" \
    POETRY_NO_INTERACTION=1 \
    POETRY_VIRTUALENVS_CREATE=false \
    PATH="$POETRY_HOME/bin:$PATH"
RUN pip3 install poetry

# Copy and install dependencies
COPY rnd/autogpt_libs /app/rnd/autogpt_libs
COPY rnd/market/poetry.lock rnd/market/pyproject.toml /app/rnd/market/
WORKDIR /app/rnd/market
RUN poetry config virtualenvs.create false \
    && poetry install --no-interaction --no-ansi

# Generate Prisma client
COPY rnd/market /app/rnd/market
RUN poetry config virtualenvs.create false \
    && poetry run prisma generate

FROM python:3.11-slim-buster AS server_dependencies

WORKDIR /app

# Copy only necessary files from builder
COPY --from=builder /app /app
COPY --from=builder /usr/local/lib/python3.11 /usr/local/lib/python3.11
COPY --from=builder /usr/local/bin /usr/local/bin
# Copy Prisma binaries
COPY --from=builder /root/.cache/prisma-python/binaries /root/.cache/prisma-python/binaries

ENV PATH="/app/.venv/bin:$PATH"

RUN mkdir -p /app/rnd/autogpt_libs
RUN mkdir -p /app/rnd/market

COPY rnd/autogpt_libs /app/rnd/autogpt_libs

COPY rnd/market /app/rnd/market

WORKDIR /app/rnd/market

FROM server_dependencies AS server

ENV DATABASE_URL=""
ENV PORT=8015

CMD ["poetry", "run", "app"]
