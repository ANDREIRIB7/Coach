# Coach de Estudos — versão Python (Streamlit)

App de estudos com cronograma automático por peso, gamificação e dashboard,
escrito 100% em Python (Streamlit). Os dados ficam guardados em **um único
arquivo JSON dentro de uma pasta do seu Google Drive** — não depende de
nenhum banco de dados que possa "hibernar" por inatividade.

Não tem tela de login: é um app pessoal, e qualquer pessoa com o link acessa
os mesmos dados (os mesmos editais, o mesmo progresso). Se um dia isso deixar
de fazer sentido (por exemplo, se for usar de mais de um lugar e quiser
restringir o acesso), dá pra adicionar uma senha simples depois.

## 1. Preparar o Google Drive (uma vez só)

1. Acesse https://console.cloud.google.com/, crie um projeto (ou use um que
   já tenha) e em **APIs e serviços → Biblioteca** ative a **Google Drive API**.
2. Em **APIs e serviços → Credenciais → Criar credenciais → Conta de serviço**,
   crie uma conta de serviço (não precisa dar nenhum papel/role especial no
   projeto).
3. Abra a conta de serviço criada → aba **Chaves** → **Adicionar chave** →
   **Criar nova chave** → formato **JSON**. Isso baixa um arquivo `.json` com
   as credenciais — guarde-o, ele não pode ser baixado de novo depois.
4. No arquivo baixado, copie o valor de `"client_email"` (algo como
   `nome@projeto.iam.gserviceaccount.com`).
5. No seu Google Drive normal, crie uma pasta (ex: "Coach de Estudos") e
   **compartilhe essa pasta** com o e-mail do passo anterior, dando permissão
   de **Editor**.
6. Pegue o **ID da pasta**: é o trecho depois de `/folders/` na URL da pasta
   quando você a abre no navegador.

## 2. Rodar localmente

1. Instale o Python 3.10+ (https://python.org) se ainda não tiver.
2. Nesta pasta, instale as dependências:
   ```
   pip install -r requirements.txt
   ```
3. Copie `.streamlit/secrets.toml.example` (se não tiver, crie o arquivo
   `.streamlit/secrets.toml`) e preencha:
   ```toml
   GDRIVE_FOLDER_ID = "o-id-da-pasta-do-passo-1.6"

   [gdrive_service_account]
   type = "service_account"
   project_id = "..."
   private_key_id = "..."
   private_key = "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
   client_email = "nome@projeto.iam.gserviceaccount.com"
   client_id = "..."
   auth_uri = "https://accounts.google.com/o/oauth2/auth"
   token_uri = "https://oauth2.googleapis.com/token"
   auth_provider_x509_cert_url = "https://www.googleapis.com/oauth2/v1/certs"
   client_x509_cert_url = "..."
   universe_domain = "googleapis.com"
   ```
   Todos esses campos (menos o `GDRIVE_FOLDER_ID`) são só uma cópia dos campos
   que já vêm no `.json` baixado no passo 1.3 — copie e cole cada um.
4. Rode:
   ```
   streamlit run app.py
   ```
   Vai abrir no navegador em `http://localhost:8501`.

## 3. Publicar online — Streamlit Community Cloud (gratuito)

1. Crie uma conta gratuita no GitHub (https://github.com) se não tiver.
2. Crie um repositório novo e suba os arquivos desta pasta (`app.py`,
   `requirements.txt`, `.streamlit/config.toml` — **não suba o
   `.streamlit/secrets.toml` com suas chaves reais**, ele já fica de fora do
   Git por padrão do Streamlit se você seguir o passo a passo deles).
3. Acesse https://share.streamlit.io, entre com sua conta do GitHub.
4. Clique em **"New app"**, escolha o repositório e o arquivo `app.py`.
5. Antes de publicar, clique em **"Advanced settings"** → **Secrets** e cole
   o mesmo conteúdo do `secrets.toml` do passo 2.3 (o `GDRIVE_FOLDER_ID` e o
   bloco `[gdrive_service_account]`).
6. Clique em **"Deploy"**. Em poucos minutos você tem uma URL do tipo
   `seu-app.streamlit.app`, acessível de qualquer celular, tablet ou notebook.

## 4. Usar seu próprio domínio (opcional)

O Streamlit Community Cloud gratuito não permite domínio próprio diretamente.
Se isso for importante, duas alternativas sem sair do Python:

- **Render.com** (tem plano gratuito e permite domínio próprio): crie um
  "Web Service" apontando para este repositório, comando de start
  `streamlit run app.py --server.port $PORT --server.address 0.0.0.0`. O
  arquivo `.streamlit/secrets.toml` também funciona nesses serviços — é só
  não versionar ele no Git e enviá-lo por outro meio (upload manual no painel
  do serviço, se ele permitir, ou um passo de build que o gera a partir de
  variáveis de ambiente).
- **Railway.app**: processo parecido, também com plano gratuito limitado.

Em ambos, depois de publicado, vá nas configurações de domínio do serviço e
aponte seu domínio próprio (registrado no Registro.br ou outro registrador).

## Sobre o link do TEC Concursos

Cada matéria guarda um link (caderno salvo ou busca encurtada) que você mesmo
gera dentro do TEC. O botão "Estudar no TEC" abre em nova guia com sua sessão
já logada. Também existe um "Tentar abrir aqui dentro do site" que tenta um
iframe — o TEC provavelmente bloqueia isso por segurança (é o padrão de sites
com login), e o próprio app avisa quando isso acontece.

## Backup

Os dados moram no arquivo `coach_estudos_state.json`, dentro da pasta do
Google Drive que você compartilhou no passo 1.5 — trocar de aparelho ou
reinstalar o Python não apaga nada. Para fazer backup, basta baixar esse
arquivo pelo próprio Google Drive (ou deixar o histórico de versões do Drive
cuidar disso automaticamente).

## Sobre a segurança dos dados (hibernação do Streamlit)

O Streamlit Community Cloud "hiberna" o app depois de um tempo sem acesso — mas isso só pausa o
processo, não roda nenhum código. Quando você acessa de novo, ele acorda, reconecta no Drive e
recarrega exatamente o que estava lá. Não existe cenário em que a hibernação em si apague algo.

O único risco real seria uma falha de rede bem no meio de um carregamento ou salvamento — por isso
o app agora:
- **Nunca** trata uma falha de leitura como "está tudo vazio, pode continuar": se o arquivo existe no
  Drive mas não consegue ser lido (rede instável), o app **para com um erro visível** em vez de seguir
  com dados em branco (o que arriscaria sobrescrever tudo no primeiro salvamento).
- Tenta salvar até 3 vezes antes de desistir, e se mesmo assim falhar, mostra um aviso bem visível
  pedindo para tentar de novo antes de fechar a aba.
- Mostra na barra lateral o horário do último salvamento bem-sucedido, e tem um botão **"💾 Salvar
  agora"** para você forçar um salvamento manual sempre que quiser ter certeza.

## Novidades desta versão

- **Nova página "Meus Editais":** tela inicial do app. Cadastre quantos editais estiver de olho,
  acompanhe a contagem regressiva até a prova e até o fim das inscrições, marque se já se inscreveu
  e se já pagou, e confirme o status de cada um (Avaliando / Vou fazer / Não vou fazer). Dali também
  dá pra abrir qualquer edital direto no Painel.
- **Nova página "Timer":** um cronômetro/timer de estudo (Pomodoro) para deixar aberto numa segunda
  tela — presets de 25/50 min de foco e 5/15 min de pausa, tempo customizado, modo cronômetro
  (contagem crescente) e um beep quando o tempo acaba. Ele roda inteiramente no navegador (não
  depende do servidor), mas não grava nada sozinho — depois de estudar, registre as questões feitas
  normalmente no card da matéria.
- **Salvamento mais robusto:** ver seção acima sobre segurança dos dados.

- **Layout novo:** o app agora usa navegação lateral (Painel, Matérias & Pesos,
  Importar planilha, Dashboard) e um visual em cartões inspirado no painel de
  referência que você enviou. O tema de cores mora em dois lugares: no CSS no
  topo de `app.py` e no arquivo `.streamlit/config.toml` (novo — não esqueça de
  subir essa pasta `.streamlit/` também, junto com o `secrets.toml` local que
  **não** deve ir pro GitHub).
- **Persistência trocada para o Google Drive:** ver seções 1 a 3 acima. Não
  tem mais tela de login — o app abre direto no painel.
- **Importação do Edital Verticalizado:** na aba "Importar planilha" agora dá
  pra subir direto o seu modelo de edital verticalizado (colunas
  *Grupo/Disciplina*, *Conteúdo Programático/Tópico* etc.), incluindo a coluna
  opcional **Link TEC** que você for adicionando a essas planilhas. O app
  reconhece automaticamente o cabeçalho e ignora as linhas de bloco (tipo
  "CONHECIMENTOS BÁSICOS - P1..."). Se a planilha tiver mais de uma aba (um
  cargo por aba), ele deixa você escolher qual importar. O formato simples
  antigo (Materia/Assunto/Peso/LinkTEC) continua funcionando como alternativa.
  Se a planilha já tiver colunas de "Questões Feitas" e "% Acertos"
  preenchidas, o app oferece a opção de importar isso como progresso da
  semana atual.
- **Modelo de planilha em .xlsx:** o botão "Baixar planilha modelo" agora gera
  um Excel formatado de verdade (título, legenda, cabeçalho colorido, colunas
  já dimensionadas, linhas de exemplo) em vez de um CSV cru.
- **Classificação Geral/Específico:** as matérias agora podem ser marcadas
  como *Geral* (conhecimentos comuns/básicos) ou *Específico* (do cargo).
  Isso aparece como uma coluna com lista suspensa no modelo de planilha, como
  coluna editável na aba Matérias & Pesos, como selo colorido ao lado de cada
  matéria no Painel, e dá pra filtrar o Painel só por Geral ou só por
  Específico. Se a planilha importada não tiver essa coluna (como no formato
  antigo do TCDF), o app tenta adivinhar pelo nome do bloco — por exemplo,
  "CONHECIMENTOS BÁSICOS" vira Geral e "CONHECIMENTOS ESPECÍFICOS" vira
  Específico automaticamente.
