# Configuração do login Google — SIGO-IFAM

## 1. Instalar a dependência

Na pasta do projeto, execute:

```powershell
npm install next-auth@4.24.11
```

Esse comando instala o NextAuth e atualiza `package-lock.json`. Depois, execute `npm run dev`.

## 2. Configurar o OAuth no Google Cloud

No projeto Google Cloud usado pela instituição:
1. Abra **APIs e serviços → Tela de permissão OAuth** e configure o aplicativo.
2. Crie um **ID do cliente OAuth** do tipo **Aplicativo da Web**.
3. Em **Origens JavaScript autorizadas**, adicione `http://localhost:3000`.
4. Em **URIs de redirecionamento autorizados**, adicione `http://localhost:3000/api/auth/callback/google`.
5. Copie o ID e o segredo do cliente para o arquivo `.env.local`. Não os envie em conversas ou repositórios.

Em produção, configure também o domínio oficial HTTPS e o URI de callback correspondente.

## 3. Variáveis de ambiente

Use `.env.example` como referência e complete o `.env.local` que já existe. Mantenha as variáveis do Google Sheets existentes e acrescente:

```env
GOOGLE_CLIENT_ID=seu-id-de-cliente
GOOGLE_CLIENT_SECRET=seu-segredo-de-cliente
NEXTAUTH_URL=http://localhost:3000
NEXTAUTH_SECRET=gere-um-segredo-aleatorio-forte
```

Para gerar um segredo no PowerShell, execute `node -e "console.log(require('crypto').randomBytes(32).toString('base64'))"` e copie o resultado para `NEXTAUTH_SECRET`.

A chave JSON da conta de serviço do Sheets deve continuar no caminho local indicado por `GOOGLE_APPLICATION_CREDENTIALS`, dentro de `.secrets/`, nunca no repositório.

## 4. Preparar o primeiro administrador DEP

O sistema não permite que alguém se torne DEP apenas escolhendo esse perfil no formulário de login. Antes do primeiro login, um responsável autorizado deve adicionar manualmente uma linha na aba `Usuarios` da planilha, com as colunas nesta ordem:

`id | nome | email | perfil | ativo`

Use um ID único, o nome autorizado, o e-mail exato da conta Google institucional, `DEP` no perfil e `Sim` em ativo. Compartilhe a planilha com a conta de serviço como Editor, como já estava configurado.

Após esse primeiro administrador entrar com Google, ele poderá acessar `/usuarios` e cadastrar os demais usuários. Escolha o perfil DEP somente para pessoas autorizadas.

## 5. Testar

1. Reinicie o servidor (`Ctrl+C` e depois `npm run dev`) após alterar `.env.local`.
2. Entre pela tela inicial com uma conta que já esteja na aba `Usuarios` e ativa.
3. Teste também uma conta não cadastrada e uma conta com `ativo` definido como `Não`: ambas devem ser recusadas.
4. Entre como DEP e abra `http://localhost:3000/usuarios` para cadastrar usuários.
5. Teste o cadastro de ocorrência com uma conta Assistente de Alunos. A API rejeita tentativas de registro feitas pelo perfil DEP.

## Observações de segurança

- O sistema consulta o perfil e o estado ativo na aba `Usuarios` durante as verificações de sessão.
- As APIs de alunos e ocorrências exigem sessão válida; criar ocorrência é permitido somente ao perfil Assistente de Alunos.
- A API de cadastro de usuários exige perfil DEP no servidor.
- Antes de usar com dados reais ou publicar na internet, revise controles de acesso, auditoria, política de retenção e segurança operacional com a instituição.
