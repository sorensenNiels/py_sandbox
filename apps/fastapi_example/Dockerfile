# ------------------------------------------------------------------------
# PY-BUILDER
# ------------------------------------------------------------------------
FROM python:3.12-bookworm as py-builder

ARG POETRY_VERSION=1.7.1
ARG APP=fastapi_example

# Install OS packages for building
RUN apt-get update && export DEBIAN_FRONTEND=noninteractive \
    && apt-get -y install --no-install-recommends build-essential

# Install Poetry
RUN pip install poetry==${POETRY_VERSION}

# Setup Poetry
ENV POETRY_NO_INTERACTION=1 \
    POETRY_VIRTUALENVS_IN_PROJECT=1 \
    POETRY_VIRTUALENVS_CREATE=1 \
    POETRY_CACHE_DIR=/tmp/poetry_cache

# Set working directory
WORKDIR /app

# Copy required files for building
COPY apps/${APP}/pyproject.toml /apps/${APP}/poetry.lock ./apps/${APP}/
COPY packages ./packages
RUN touch ./apps/${APP}/README.md

RUN cd ./apps/${APP} && poetry install --no-dev --no-interaction --no-ansi --no-root && rm -rf $POETRY_CACHE_DIR

# ------------------------------------------------------------------------
# PY-RUNTIME
# ------------------------------------------------------------------------
FROM python:3.12-alpine3.19 as py-runtime

ARG WORK_DIR=app
ARG PYTHON_ENV=development
ARG APP=fastapi_example

# Setup path to virtual environment
ENV VIRTUAL_ENV=/${WORK_DIR}/apps/${APP}/.venv 
ENV PATH="${VIRTUAL_ENV}/bin:$PATH"

# Create the app user and home folder
RUN addgroup -S app && adduser -S app -G app

WORKDIR /${WORK_DIR}

# Create data folder
RUN mkdir -p /${WORK_DIR}/apps/${APP}/data && chown -R app:app /${WORK_DIR}/apps/${APP}/data

# Get the dependencies from build step
COPY --from=py-builder ${VIRTUAL_ENV} ${VIRTUAL_ENV}
COPY --from=py-builder /app/packages ./packages

WORKDIR /${WORK_DIR}/apps/${APP}

# Provide required files for runtime
COPY apps/${APP} ./

COPY apps/${APP}/bin/ ./bin

USER app

EXPOSE 8000
VOLUME [ "${WORK_DIR}/${APP}/data" ]

CMD [ "./bin/takeoff.sh" ]
