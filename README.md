# Guess Game - Docker Compose

Estrutura em Docker Compose para o jogo de adivinhação (https://github.com/fams/guess_game), com backend Flask, banco Postgres e NGINX como proxy reverso servindo o frontend React e balanceando carga entre múltiplas instâncias do backend.

## Arquitetura

Três serviços:

- **db**: Postgres 16. Guarda os dados do jogo. Os dados ficam em um volume nomeado (`pgdata`), separado do container, garantindo persistência mesmo que o container seja recriado.
- **backend**: aplicação Flask. Sobe em 3 instâncias (réplicas). Não expõe porta para o host; só é acessível pela rede interna do Compose. Cria a tabela do jogo automaticamente na primeira conexão com o banco.
- **nginx**: serve os arquivos estáticos do React e atua como proxy reverso. As chamadas de API (`/create`, `/guess/`, `/health`) são encaminhadas para as instâncias do backend. É o único serviço com porta publicada para o host.

Todos os serviços usam `restart: always`. Se qualquer container falhar, ele é reiniciado automaticamente.

## Balanceamento de carga

O backend roda com 3 réplicas. O NGINX resolve o nome `backend` pelo DNS interno do Docker (`127.0.0.1:11` / `127.0.0.11`) a cada requisição, distribuindo as chamadas entre as réplicas ativas. Aumentar ou reduzir o número de instâncias não exige mudança na configuração do NGINX.

O frontend é buildado com `REACT_APP_BACKEND_URL` vazio, então o navegador chama a mesma origem do NGINX (ex: `/create`), que faz o proxy para o backend. Isso evita CORS e mantém um único ponto de entrada.

## Pré-requisitos

- Docker
- Docker Compose

## Como rodar

```
git clone https://github.com/GabriRibeiro/guess_game_01.git
cd guess_game_01
docker compose up -d --build
```

Acesse:

```
http://localhost:8080
```

Para verificar os serviços:

```
docker compose ps
```

Para ver os logs:

```
docker compose logs -f
```

Para parar:

```
docker compose down
```

Para parar e apagar os dados do banco:

```
docker compose down -v
```

## Ajustar o número de instâncias do backend

O padrão é 3. Para mudar sem editar arquivos:

```
docker compose up -d --scale backend=5
```

## Atualização de componentes

Cada componente é atualizado trocando a versão da imagem, sem alterar o código.

- **Backend**: altere a tag em `image: guess-backend:1.0.0` no `docker-compose.yml` e rode `docker compose up -d --build backend`.
- **Frontend/NGINX**: altere a tag em `image: guess-frontend:1.0.0` e rode `docker compose up -d --build nginx`.
- **Banco**: altere a tag da imagem em `image: postgres:16-alpine` para a versão desejada e rode `docker compose up -d db`. O volume `pgdata` preserva os dados.

## Estrutura do repositório

```
guess_game_01/
├── docker-compose.yml
├── Dockerfile.backend
├── Dockerfile.frontend
├── nginx/
│   └── default.conf
├── run.py
├── guess/
├── repository/
├── frontend/
└── requirements.txt
```
