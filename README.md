# SIGO — Gestão de Ocorrências (IFAM)

Sistema web (instalável no celular) para registrar e acompanhar ocorrências de alunos do Instituto Federal: atraso (tolerância de 15 min) e falta de item do uniforme (camisa, calça preta, calçado fechado). Substitui a ficha manual. Segue as Resoluções nº 57 e nº 94 da instituição (a regra exata de contagem e de penalidade deve ser confirmada nelas, não presumida pelo código).

## Situação atual
- **Front-end (Pessoa 1): pronto como demonstração.** Next.js + React + TypeScript + Tailwind v4. Todos os dados são de teste e ficam na memória: ao recarregar a página, volta tudo ao início.
- **Back-end e banco (Pessoas 2 e 3): a fazer.** Falta API, autenticação, banco e ligar cada ação do front.

## Como rodar
```
npm install
npm run dev     # abre em http://localhost:3000
```
Requer Node.js LTS. No PowerShell, se os scripts forem bloqueados: `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`.

Todo o front está em `app/page.tsx` (telas e dados de teste) e `app/globals.css` (estilos).

## Perfis e permissões (como o front já se comporta)
| Perfil | Vê e faz |
|---|---|
| **Assistente de alunos / Inspetor** (celular ou notebook) | Dashboard, Registrar ocorrência, Alunos, Ocorrências (pode **editar** ocorrência: aluno, data, horário, tipo, item, observação). **Não vê** alertas, "requer análise", nível disciplinar nem progressão. |
| **DEP** (Departamento de Ensino) | Tudo acima (menos registrar) + Alertas, Progressão disciplinar, Assistência Social, Relatórios, Configurações, Usuários (cadastra e edita usuários). |

Esconder telas no front **não protege os dados**: a API deve checar o perfil e não devolver alertas nem penalidades ao assistente de alunos.

## Regras de negócio
- Campos obrigatórios da ocorrência: data, turma, nome do aluno e tipo (atraso ou item do uniforme).
- A **data é a data em que o fato aconteceu** e pode ser anterior à data do registro (não pode ser futura). Guardar também quando foi registrada e quem registrou.
- Toda edição de ocorrência deve guardar quem editou, quando e o que mudou (auditoria).
- Ao acumular **3 ocorrências**, gerar alerta ao DEP. O alerta **não aplica punição automática**: ocorrência → contagem → alerta → análise do DEP → medida disciplinar.
- Progressão disciplinar: advertência verbal → advertência por escrito → notificação/convocação dos responsáveis. Nenhuma etapa avança sem registro da ação.
- Dificuldade financeira para o uniforme: encaminhar à Assistência Social antes de qualquer decisão disciplinar (sem penalização sumária). Dados socioassistenciais com acesso restrito.
- Histórico mensal por aluno: quantidade de infrações por mês e a penalidade aplicada em cada mês.

## Telas
Login (escolha de perfil), Dashboard, Registrar ocorrência (busca de aluno com autocomplete), Alunos, Perfil do aluno (linha do tempo + histórico por mês), Ocorrências (com edição), Alertas, Progressão disciplinar, Notificação aos responsáveis, Assistência Social e detalhes do acompanhamento, Relatórios (visualizar, exportar CSV/Excel e PDF pela impressão), Configurações (atraso, progressão, notificações), Usuários.

## Sugestão de modelo de dados (a combinar entre os três)
- `usuario`: id, nome, email, senha_hash, perfil (inspetor | dep | assistencia_social | admin), ativo
- `turma`: id, nome, curso
- `aluno`: id, nome, matricula, turma_id
- `ocorrencia`: id, aluno_id, registrado_por (usuario_id), data_fato, horario, tipo (uniforme | atraso), item, observacao, situacao, criado_em, editado_em, editado_por
- `alerta`: id, aluno_id, criado_em, status
- `medida_disciplinar`: id, aluno_id, etapa (verbal | escrita | notificacao), data, responsavel_id, observacao
- `notificacao_responsavel`: id, aluno_id, data, responsavel_notificado, meio, status (pendente | enviada | recebida | concluida)
- `encaminhamento_social`: id, aluno_id, data, motivo, status (aguardando_triagem | em_acompanhamento | resolvido | encerrado)
- `acompanhamento_social`: id, encaminhamento_id, data, registro, autor_id
- `configuracao`: horario_entrada, tolerancia_minutos, ocorrencias_para_alerta, etapas ativas, regras de notificação

## Endpoints mínimos sugeridos
- `POST /login`
- `GET /alunos?busca=mar` (autocomplete: nome ou matrícula; o front hoje filtra uma lista fixa)
- `POST /ocorrencias`, `GET /ocorrencias`, `PATCH /ocorrencias/:id`
- `GET /alertas` (só DEP), `POST /medidas`, `POST /notificacoes`
- `GET/POST /encaminhamentos`, `POST /encaminhamentos/:id/acompanhamentos`
- `GET /relatorios?de=&ate=&turma=&tipo=`
- `GET/POST/PATCH /usuarios` (só DEP)
- `GET/PUT /configuracoes` (só DEP)

**Atenção:** o front usa nomes de campo em inglês/abreviados no código (`date`, `time`, `student`, `tipo`, `item`, `obs`, `by`, `status`). Combinem os nomes finais da API antes de ligar, para não gerar incompatibilidade.

## Pendências do front
- Ligar cada ação à API (hoje tudo é dado de teste).
- "Cadastrar aluno", filtros, paginação e o botão "Detalhes" do dashboard ainda são visuais.
- O Perfil do aluno ainda mostra sempre o mesmo aluno de exemplo.
- Testar no celular e publicar (Vercel, plano Hobby, projeto não comercial).

## Fluxo de trabalho em equipe
- Branch `main` protegida na prática: ninguém empurra direto. Cada um trabalha em sua branch (`backend`, `database`, `feature/...`) e abre Pull Request.
- Integrar em partes pequenas e com frequência, testando front + API + banco juntos.
- O GitHub é a fonte oficial do código. O ChatGPT ou o Claude ajudam a escrever, mas o projeto não fica guardado neles.
