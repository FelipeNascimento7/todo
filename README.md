# ToDo — Express + SQLite + Login

Versão simples do app ToDo usando:

- Node.js + Express
- SQLite em `/app/data/app.db`
- Login por usuário e senha
- Sessão via cookie HttpOnly/Secure
- Docker
- Traefik já existente na VPS
- Domínio configurado no Compose: `todo.appdoseujeito.online`

## Estrutura

```text
todo-sqlite-docker/
├── compose.yaml
├── Dockerfile
├── package.json
├── server.js
├── .env.example
└── public/
    ├── index.html
    └── login.html
```

## Persistência

O SQLite fica em um volume Docker chamado:

```text
todo_data
```

O container pode ser recriado sem apagar o banco.

## Configuração inicial

Na VPS:

```bash
cd /opt/apps/todo
cp .env.example .env
nano .env
```

Altere obrigatoriamente:

```env
APP_USERNAME=felipe
APP_PASSWORD=uma-senha-forte-com-mais-de-10-caracteres
SESSION_SECRET=uma-chave-aleatoria-com-pelo-menos-32-caracteres
```

Para gerar `SESSION_SECRET`:

```bash
openssl rand -hex 32
```

Proteja o arquivo:

```bash
chmod 600 .env
```

## DNS

Crie na Hostinger:

```text
Tipo: A
Nome: todo
Valor: 179.199.136.166
```

Confirme:

```bash
dig +short todo.appdoseujeito.online
```

## Subir

```bash
sudo docker compose config
sudo docker compose up -d --build
sudo docker compose ps
sudo docker logs --tail 100 todo
```

Abra:

```text
https://todo.appdoseujeito.online
```

## Trocar senha

Edite:

```bash
nano .env
```

Altere `APP_PASSWORD` e recrie o container:

```bash
sudo docker compose up -d --build
```

O hash armazenado no SQLite será atualizado no boot.

## Backup simples do SQLite

O volume não publica a porta do banco. Para gerar uma cópia do diretório de dados:

```bash
mkdir -p ~/backups
sudo docker run --rm \
  -v todo-sqlite-docker_todo_data:/data:ro \
  -v "$HOME/backups:/backup" \
  alpine \
  sh -c 'tar czf /backup/todo-data-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .'
```

> O nome real do volume pode variar. Veja com `sudo docker volume ls`.

## Observações

- Não existe porta pública de SQLite.
- O frontend usa planejamento por data e horário.
- O estado inteiro é salvo no SQLite por usuário.
- O Google Drive foi removido desta versão para manter a arquitetura simples.
- Exportação/importação CSV continuam disponíveis.

## Planejamento e execução

Esta versão inclui:

- duração configurável de cada bloco de execução;
- campos **Hora início** e **Hora fim** nas tarefas agendadas;
- cálculo automático de **Blocos necessários** e ajuste do intervalo ao salvar para completar blocos inteiros;
- validação do tempo disponível do dia e de conflitos de horário antes do agendamento;
- duração acima de 60 minutos exibida em horas e minutos;
- tela **Execução** mostrando a próxima tarefa pendente por data e horário;
- constância dos últimos 7 dias com percentual e relação `concluídas/programadas`;
- tela **Semana** sempre abrindo na semana atual, com horários e intervalos livres entre tarefas;
- recorrências diária, semanal, quinzenal e mensal para tarefas comuns;
- estrutura nativa **ROTINA → DIA DA SEMANA → tarefa**, com os dois primeiros níveis protegidos;
- tarefas da ROTINA com recorrência semanal ou quinzenal e reagendamento automático após a conclusão;
- projeção das tarefas da ROTINA na aba **Semana** em todas as semanas correspondentes à recorrência, mantendo o dia fixo e contabilizando o horário como ocupado;
- navegação mobile por menu de abas no rodapé, com o nome da aba centralizado.

Tarefas salvas por versões anteriores continuam compatíveis com o estado atual.
