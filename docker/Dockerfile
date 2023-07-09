FROM python:3.11-alpine as base

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    POETRY_HOME=/etc/poetry \
    PROJECT_DIR=/app
ENV PATH="$POETRY_HOME/bin:$PATH"

RUN apk update \
    && apk add --no-cache\
        curl=8.1.2-r0 \
        && rm -rf /var/cache/apk/*

FROM base as builder

WORKDIR $PROJECT_DIR

SHELL ["/bin/ash", "-o", "pipefail", "-c"]
RUN curl -sSL https://install.python-poetry.org | python3 -

COPY poetry.lock pyproject.toml ./
RUN poetry export --only main --without-hashes > requirements.txt


FROM base as prod

WORKDIR $PROJECT_DIR

RUN addgroup -g 10001 python \
    && adduser -D -u 10000 -G python -s /bin/ash python

COPY --chown=python:python --from=builder $PROJECT_DIR $PROJECT_DIR
COPY --chown=python:python . .

RUN pip install --no-cache-dir -r requirements.txt \
    && rm -rf /root/.cache/pip

WORKDIR /$PROJECT_DIR/src

USER python
EXPOSE 8000

ENTRYPOINT ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]