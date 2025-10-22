1. Serviços Principais (Core)

Estas são as funcionalidades centrais que dão suporte a ambas as APIs.

    Autenticação e Autorização:

        Registrar novos usuários (User).

        Autenticar usuários (ex: via email/senha, retornando um token JWT).

        Proteger rotas (Middleware) para garantir que apenas usuários autenticados possam acessar a API GraphQL.

        Gerenciamento de permissão (ex: verificar se um usuário pertence à Organization correta antes de permitir que ele edite um QrCode).

    Geração de QR Code:

        Um serviço/módulo interno que recebe uma string (a URL de redirecionamento, ex: https://qraft.com/r/aB3xY) e retorna uma imagem (SVG ou PNG) do QR Code.

        Isso pode ser uma Query no GraphQL (getQrCodeImage(id: "...")) que retorna a imagem em Base64, ou um endpoint REST simples (GET /api/qrcode/:id/image) que retorna o arquivo de imagem.

    Geração de Código Curto (Short Code):

        Função interna que gera uma string única (ex: aB3xY) para cada novo QrCode.

        Deve garantir que o código gerado ainda não exista no banco de dados.

2. API de Gerenciamento (GraphQL - /graphql)

Esta é a API "privada", usada pelo seu dashboard de administração para gerenciar tudo.

    User & Organization (CRUD):

        Mutations:

            login(email, password): Retorna um token de autenticação.

            register(name, email, password): Cria um novo User e uma Organization padrão para ele.

            inviteUserToOrganization(email): (Funcionalidade futura) Convida um novo membro.

        Queries:

            me(): Retorna os dados do usuário atualmente logado.

            myOrganization(): Retorna os dados da organização do usuário.

    QrCode (CRUD):

        Mutations:

            createQrCode(name: String!, destinationUrl: String!):

                Valida a destinationUrl.

                Gera um short_code único.

                Salva o novo QrCode no banco, associado ao owner_id e organization_id do usuário.

                Retorna o objeto QrCode completo.

            updateQrCode(id: ID!, name: String, destinationUrl: String): Atualiza os dados de um QR Code existente.

            deleteQrCode(id: ID!): Remove um QR Code.

        Queries:

            getQrCode(id: ID!): Busca um QrCode específico pelo seu ID.

            listQrCodes(page: Int, filter: String): Lista todos os QR Codes da organização do usuário, com paginação e filtro.

    Analytics (Leitura):

        Queries:

            getQrCodeAnalytics(id: ID!, timeRange: String!): Busca todos os Scans de um QR Code específico.

                Retorna contagem total de scans.

                Retorna dados agregados para gráficos (ex: scans por dia).

                Retorna dados agregados por geografia (país/cidade).

                Retorna dados agregados por dispositivo (User Agent).

            getDashboardSummary(): Retorna estatísticas rápidas para a página inicial (ex: total de QRs, total de scans no mês).

3. API de Redirecionamento (REST - /r/:short_code)

Esta é a API "pública", otimizada para velocidade. É o endpoint para o qual o QR Code físico aponta.

    Endpoint: GET /r/:short_code

        Propósito: Redirecionar o usuário para a URL de destino e registrar o scan.

        Processo (Síncrono):

            Recebe a requisição (ex: GET /r/aB3xY).

            Extrai o short_code ("aB3xY") da URL.

            Faz uma busca no banco de dados: SELECT destination_url, id FROM QrCode WHERE short_code = "aB3xY".

            Se não encontrar: Retorna uma resposta HTTP 404 Not Found.

            Se encontrar: a. Pega a destination_url (ex: "https://meusite.com") e o qr_code_id. b. Coleta os dados do scan da requisição: User-Agent, Endereço IP (anonimizado/hash para LGPD). c. Grava no banco (Síncrono): Executa um INSERT INTO Scan (qr_code_id, scanned_at, ip_hash, user_agent) VALUES (...). d. Redireciona: Retorna uma resposta HTTP 302 Found (ou 301) com o cabeçalho Location: https://meusite.com.
