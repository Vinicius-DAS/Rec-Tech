# Deploy no Render

Passo a passo pra colocar o Rec-Tech no ar, gratuito.

## Por que Render + Neon (não Azure, não o Postgres do próprio Render)

- O link antigo do Azure (`rec-tech.azurewebsites.net`) não existe mais, e os segredos de deploy do Azure não estão configurados neste fork — não dá pra usar o workflow `prod_rec-tech.yml` como está.
- **O Postgres gratuito do próprio Render expira em 30 dias.** Uso o [Neon](https://neon.tech) pro banco — plano gratuito sem expiração.

## 0. Antes de tudo: gerar uma chave nova do Google Maps

Este projeto tinha uma chave de API do Google Maps **exposta no código-fonte**, usada pela funcionalidade de "melhor rota" (coletores). Ela foi removida do código e deve ser tratada como comprometida — **não reutilize a antiga**.

1. Crie um projeto no [Google Cloud Console](https://console.cloud.google.com/) (ou use um existente).
2. Ative a **Directions API**.
3. Crie uma **API key** nova em "Credenciais". Restrinja por API (só Directions) e, se possível, por IP/referrer.
4. Guarde essa chave nova — vai usar no passo 2.

## 1. Criar o banco no Neon

1. Crie uma conta em [neon.tech](https://neon.tech) (dá pra usar login do GitHub).
2. Crie um projeto novo — qualquer nome, região "US East 2 (Ohio)" (a mesma que o Render usa por padrão — bom pra latência banco↔servidor).
3. Em **Connect**, pegue a connection string e separe em 4 valores (não use a string inteira): **Database name**, **Host**, **Role/User**, **Password**.

## 2. Criar o serviço no Render

1. Crie uma conta em [render.com](https://render.com) (dá pra usar login do GitHub).
2. **New → Blueprint**, selecione o repositório `Vinicius-DAS/Rec-Tech` (branch `main`).
3. O Render detecta o `render.yaml` na raiz do repo automaticamente.
4. Quando pedir os 5 valores obrigatórios (`DBNAME`, `DBHOST`, `DBUSER`, `DBPASS`, `GOOGLE_MAPS_API_KEY`), cola os valores dos passos 0 e 1.
5. Confirma a criação — o Render builda e sobe o serviço automaticamente.

O `render.yaml` já cuida do resto: `SECRET_KEY` gerado automaticamente, `DEBUG=0`, migrações aplicadas a cada deploy, arquivos estáticos coletados no build.

## 3. Popular com dados de demonstração

No painel do Render, abra a aba **Shell** do serviço (obs: só disponível em planos pagos — se estiver no free, rode o comando abaixo do seu próprio terminal, apontando pro banco de produção, como fizemos no Conecta Cesar) e rode:

```bash
python manage.py create_objects
```

Isso cria os bairros, lixeiras e as 3 contas de demonstração — usuários **admin**, **coletor**, **cliente**, todas com senha **123**.

## 4. Depois de estar no ar

Volta aqui e me manda a URL que o Render gerou (tipo `rec-tech.onrender.com`) que eu atualizo o link do projeto no portfólio.

## Nota sobre o "spin down"

No plano gratuito, o serviço "dorme" depois de um tempo sem acesso e demora uns 30-60 segundos pra acordar na primeira visita depois disso.
