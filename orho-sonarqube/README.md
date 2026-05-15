# orho-sonarqube

SonarQube Community Build as an Umbrel community app.

## Estrutura de arquivos

Coloque esta pasta `orho-sonarqube/` na raiz do seu repositório de community store
(mesmo nível dos exemplos `orho-hello-world` e `orho-nginx`), junto do
`umbrel-app-store.yml` já existente.

```
umbrel-community-store/
├── umbrel-app-store.yml      # id: orho / name: orho
├── orho-hello-world/
├── orho-nginx/
└── orho-sonarqube/           # esta pasta
    ├── umbrel-app.yml
    ├── docker-compose.yml
    └── README.md
```

> O `id` do app **precisa** começar com o `id` da store (`orho`), por isso o app
> se chama `orho-sonarqube`. Se mudar o id da sua store, renomeie a pasta e
> ajuste o campo `id:` em `umbrel-app.yml`.

## Como instalar no Umbrel

1. Faça commit/push dos arquivos no seu repositório GitHub.
2. No umbrelOS, abra a App Store → ícone do usuário no topo direito →
   **Community App Stores** → **Add a community app store**.
3. Cole a URL do seu repositório (ex: `https://github.com/rauppcs/umbrel-community-store`).
4. O app **SonarQube** vai aparecer na sua store recém-adicionada. Clique em **Install**.

## Primeiro acesso

- Abra o app pelo dashboard do Umbrel (ele usa o `app_proxy` na porta padrão).
- Credenciais padrão: usuário `admin`, senha `admin`.
- Você **vai ser obrigado** a trocar a senha no primeiro login.

> ⏱️ A primeira inicialização leva de 2 a 5 minutos enquanto o Elasticsearch
> embutido e o banco H2 sobem. Se aparecer "Bad Gateway", aguarde e atualize.

## Requisitos

- **RAM**: mínimo 2 GB livres, idealmente 4 GB+. Raspberry Pi 4 com 4 GB ou mais.
- **Arquitetura**: amd64 ou arm64. **Não roda em arm/v7 (Pi de 32 bits)**.
- **Disco**: ~2 GB para a imagem + o que seus projetos analisarem.

## Observações sobre o docker-compose

Em relação ao compose original do material de apoio, fiz as adaptações padrão do
Umbrel para um app de community store:

1. **Sem `ports:` no container do SonarQube.** No Umbrel todo tráfego externo passa
   pelo serviço `app_proxy`, que pega as envs `APP_HOST` e `APP_PORT` e expõe o
   app na porta declarada no `umbrel-app.yml` (`port: 9000`).

2. **`container_name` removido.** O Umbrel gera o nome do container a partir do
   `id` do app + nome do service (`orho-sonarqube_server_1`). Definir
   `container_name` manualmente quebra o `app_proxy`.

3. **Volumes apontando para `${APP_DATA_DIR}/data/...`** em vez de volumes nomeados
   do Docker. Assim os dados ficam dentro de `~/umbrel/app-data/orho-sonarqube/data/`,
   o que permite backup, mover entre máquinas, e os dados sobrevivem a um
   reinstall do app.

4. **`networks:` removido.** O Umbrel cria uma rede dedicada por app
   automaticamente e injeta o `app_proxy` nela; declarar `sonarnet` manualmente
   quebraria a comunicação com o proxy.

5. **`SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true` mantido** — necessário para o
   Elasticsearch embutido subir sem reclamar do `vm.max_map_count` do host
   (algo que o Umbrel não nos deixa controlar via sysctls no app).

6. **`ulimits` para `nofile`** — recomendação da própria documentação do
   SonarQube para o Elasticsearch embutido.

## Upgrade do SonarQube

A imagem está fixada em `sonarqube:community` (Community Build, atualizada
mensalmente pela SonarSource). Para atualizar:

- Atualize o campo `version:` em `umbrel-app.yml` para acompanhar a Community Build.
- Faça commit/push.
- No umbrelOS, vá em **Settings → App Store → Update** ou desinstale/reinstale o app.
  Os dados em `${APP_DATA_DIR}/data/sonarqube_data` são preservados.

## Produção / banco PostgreSQL

O H2 embutido é só para avaliação — a própria SonarSource não recomenda para uso
sério. Para evoluir para produção, adicione um serviço `db: postgres:15` no
compose, monte volume em `${APP_DATA_DIR}/data/postgres`, e exporte as envs
`SONAR_JDBC_URL`, `SONAR_JDBC_USERNAME`, `SONAR_JDBC_PASSWORD` no serviço
`server`. Se quiser, abra uma issue que eu te ajudo a montar essa variante.
