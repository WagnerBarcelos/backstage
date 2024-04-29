# [Backstage](https://backstage.io)

Esse são os passos que foram seguidos para a execução do projeto:

Utilize os seguintes guias como base:

1. Rodando o Backstage com Docker:

- Instale o Backstage executando o comando npx @backstage/create-app@latest --skip-install
- Defina um nome para a aplicação
- Navegue até a pasta da aplicação e execute o comando yarn install para instalar as dependências

2. Preparando o build do Backstage:

- Execute o comando yarn install --frozen-lockfile
- Prepare os tipos com o comando yarn tsc
- Execute o comando yarn build:backend

3. Ajustando o Dockerfile do Backstage:

- Abra o projeto no vscode
- Acesse o Dockerfile do backend em packages > backend > Dockerfile
- Substitua todo o conteúdo pelo código abaixo:

``` FROM node:18-bookworm-slim

# Install isolate-vm dependencies, these are needed by the @backstage/plugin-scaffolder-backend.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends python3 g++ build-essential && \
    yarn config set python /usr/bin/python3

# Install sqlite3 dependencies. You can skip this if you don't use sqlite3 in the image,
# in which case you should also move better-sqlite3 to "devDependencies" in package.json.
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
    --mount=type=cache,target=/var/lib/apt,sharing=locked \
    apt-get update && \
    apt-get install -y --no-install-recommends libsqlite3-dev

# From here on we use the least-privileged `node` user to run the backend.
USER node

# This should create the app dir as `node`.
WORKDIR /app

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV development

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn install --frozen-lockfile --production --network-timeout 300000

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml"] ```

```
5. Rodando no Docker:

- Execute o comando docker image build . -f packages/backend/Dockerfile --tag backstage --no-cache (utilize o comando --no-cache para não reutilizar imagens anteriores)
- Execute o container com o comando docker run -it -p 7007:7007 backstage

6. Após concluir esses passos, você pode acessar o Backstage em http://localhost:7007

# Evidências da execução da aplicação



![image](https://github.com/WagnerBarcelos/backstage/assets/99257595/9e76fae1-f63c-48e3-b218-a5787170ad76)



