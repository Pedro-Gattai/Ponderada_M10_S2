# Ponderada - Backstage
## Processo de compilação e Execução
instalando o backstage:
<img src="./Captura de tela de 2024-05-05 17-00-29.png"> </img>
baixando as dependencias:
<img src="./Captura de tela de 2024-05-05 17-14-53.png"></img>
Preparando para o build do backstage:
<img src="./Captura de tela de 2024-05-05 17-15-06.png"></img>
Preparando o containber do backstage:
<img src="./Captura de tela de 2024-05-05 17-21-27.png"></img>
Conatiner do backstage rodando:
<img src="./Captura de tela de 2024-05-05 17-25-29.png"></img>

## Passo a passo para rodar o backstage

### Foi usado os seguintes guias base:

https://backstage.io/docs/getting-started/

https://backstage.io/docs/deployment/docker/

### Como rodar o backstage com Docker

Instalar o backstage:

Rodar o comando npx @backstage/create-app@latest --skip-install

Definir um nome para a aplicação

Após criar a aplicação, navegue até a pasta da aplicação e rode o comando yarn install
Caso o seu yarn esteja desatualizado, rodar o comando npm install --global yarn

### Preparar o build do backstage:

Rode o comando yarn install --frozen-lockfile

Preparar os types com o comando yarn tsc

Rodar o comando yarn build:backend

### Ajustar o Dockerfile do backstage:

Abra o projeto no vscode

Acesse o Dockerfile do backend  em packages > backend > Dockerfile
Substitua tudo pelo código abaixo (retirado do guia de build com docker e ajustado)
```
FROM node:18-bookworm-slim

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
# If it is instead created as `root` then the `tar` command below will
# fail: `can't create directory 'packages/': Permission denied`.
# If this occurs, then ensure BuildKit is enabled (`DOCKER_BUILDKIT=1`)
# so the app dir is correctly created as `node`.
WORKDIR /app

# This switches many Node.js dependencies to production mode.
ENV NODE_ENV development

# Copy repo skeleton first, to avoid unnecessary docker cache invalidation.
# The skeleton contains the package.json of each package in the monorepo,
# and along with yarn.lock and the root package.json, that's enough to run yarn install.
COPY --chown=node:node yarn.lock package.json packages/backend/dist/skeleton.tar.gz ./
RUN tar xzf skeleton.tar.gz && rm skeleton.tar.gz

RUN --mount=type=cache,target=/home/node/.cache/yarn,sharing=locked,uid=1000,gid=1000 \
    yarn install --frozen-lockfile --production --network-timeout 300000

# Then copy the rest of the backend bundle, along with any other files we might want.
COPY --chown=node:node packages/backend/dist/bundle.tar.gz app-config*.yaml ./
RUN tar xzf bundle.tar.gz && rm bundle.tar.gz

CMD ["node", "packages/backend", "--config", "app-config.yaml"]
```
PS: Esse Dockerfile não é o mesmo que vem por Default, é o do guia. Alterei também a linha 28, substituindo ENV NODE_ENV production para ENV NODE_ENV development como pode ser visto nesse Dockerfile.

### Rodar no docker:
Rodar o comando docker image build . -f packages/backend/Dockerfile --tag backstage --no-cache  (comando --no-cache para não reutilizar imagens anteriores)

Executar o container com o comando docker run -it -p 7007:7007 backstage

Após concluir, abrir http://localhost:7007

## Backstage em execução:
<img src="./Captura de tela de 2024-05-05 17-25-46.png"></img>

## Lições Aprendidas com a Implementação do Backstage
### Introdução
Este documento resume os conhecimentos e insights adquiridos ao configurar e explorar a ferramenta Backstage em um ambiente Docker. Backstage é uma plataforma de desenvolvimento usada para gerenciar softwares e serviços dentro de uma empresa.

## Configuração e Implementação
### Desafios de Configuração
Durante a configuração do Backstage, enfrentei desafios relacionados a:

- Gerenciamento de dependências: A complexidade de gerenciar diversas dependências, incluindo as bibliotecas específicas necessárias para o Docker e o Node.js.
- Dockerização: Aprendi sobre a importância de construir imagens Docker leves e eficientes para agilizar o desenvolvimento e a implantação.
### Insights Técnicos
- Automação com Docker: A capacidade de Dockerizar uma aplicação como o Backstage ilustrou o poder da automação e da padronização em ambientes de desenvolvimento.
- Arquitetura de Microserviços: O Backstage é construído com uma arquitetura de microserviços, o que facilita a escalabilidade e a manutenção.
## Funcionalidades e Utilização
### Catálogo de Serviços
O Catálogo de Serviços é uma funcionalidade central do Backstage. Aprendi como:

- Integrar e Gerenciar Componentes: Gerenciar serviços, websites, e bibliotecas de forma centralizada oferece uma visão clara da infraestrutura de software.
- Metadados e Plugins: A importância de metadados ricos para melhorar a descoberta e gestão de componentes.
### Automação e Orquestração
- Automação de Tarefas: Descobri como o Backstage pode automatizar tarefas de rotina, como configurações de CI/CD e gestão de incidentes.
- Plugins: Explorar os diversos plugins disponíveis proporcionou insights sobre como expandir as funcionalidades do Backstage para atender necessidades específicas.
## Conclusões
### Impacto Organizacional
- Visibilidade e Controle: A implementação do Backstage pode oferecer uma visão holística de todas as operações de software, o que melhora a transparência e o controle dentro de uma organização.
- Colaboração e Produtividade: Ferramentas como o Backstage podem aumentar significativamente a colaboração entre equipes técnicas e não técnicas, resultando em uma maior produtividade e alinhamento estratégico.

## Conhecimento adicionais
Com uma ferramenta como o Backstage, é possível acelerar o desenvolvimento da equipe por conta de suas automações e facilidades disponibilizadas em processos como o de CI/CD. Também é possível trazer diversas vantagens para os desenvolvedores, como facilidades para executar processos existentes dentro do projeto/organização. Esses processos, que apenas membros mais antigos da equipe conhecem ou processos que não têm documentação, ajudam também a garantir que não tenhamos códigos duplicados. Conseguimos também fazer com que os desenvolvedores da equipe consigam consumir outros serviços, como banco de dados e sistemas de mensageria já existentes dentro da empresa, de forma mais simplificada e ver as tecnologias e plataformas usadas dentro do projeto/organização.

