# Install dependencies while network access is available.
FROM node:22-bookworm

ENV DEBIAN_FRONTEND=noninteractive \
    CI=true

RUN apt-get update && apt-get install -y --no-install-recommends \
      ca-certificates git \
    && rm -rf /var/lib/apt/lists/*

COPY . /src
WORKDIR /src

# Use the pnpm version pinned in package.json "packageManager".
RUN corepack enable && pnpm install --frozen-lockfile

# Call tsc directly. "pnpm run build" also runs eslint --fix and prettier
# --write, which would change the checked-out source.
RUN pnpm exec tsc

RUN pnpm exec vitest run src/tests/unit || echo "WARNING: unit tests reported failures"

CMD ["bash"]
