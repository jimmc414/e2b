# This stage builds playground
FROM node:18 as playground

WORKDIR /playground

COPY playground/package.json playground/package-lock.json ./
RUN npm ci

COPY playground/tsconfig.json ./
COPY playground/tsoa.json ./
COPY playground/scripts/build.js ./scripts/build.js
COPY playground/src ./src
RUN npm run build


# This is the final image that also installs python deps
FROM nikolaik/python-nodejs

ENV PYTHONFAULTHANDLER=1 \
    PYTHONHASHSEED=random \
    PYTHONUNBUFFERED=1

ENV PIP_DEFAULT_TIMEOUT=100 \
    PIP_DISABLE_PIP_VERSION_CHECK=1 \
    PIP_NO_CACHE_DIR=1

# RUN pip install poetry==1.3.2

RUN poetry config virtualenvs.in-project true

WORKDIR /app
COPY pyproject.toml poetry.lock ./
RUN --mount=type=cache,target=/home/.cache/pypoetry/cache \
    --mount=type=cache,target=/home/.cache/pypoetry/artifacts \
    poetry install --without dev --no-root --no-ansi --no-interaction

COPY . ./
RUN rm -rf ./playground

WORKDIR /playground
COPY --from=playground ./playground/node_modules ./node_modules
COPY --from=playground ./playground/lib ./lib

WORKDIR /app

CMD exec ./start.sh
