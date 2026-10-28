# Sleep to Train

[Abrir aplicativo](https://san2014.github.io/sleep-to-train/) · [Repositório no GitHub](https://github.com/san2014/sleep-to-train)

MVP pessoal para registrar informações diárias de sono e disposição para treino, gerando mensagens que podem ser copiadas para chats de acompanhamento.

O aplicativo funciona em uma única página HTML, em português, sem conta, servidor ou dependências externas. Os dados são preenchidos manualmente; não há integração com relógios, aplicativos de saúde ou serviços de inteligência artificial.

## Funcionalidades atuais

- Registro de sono: horário de despertar, duração total, sono profundo, REM e leve, despertares, tempo acordado, qualidade percebida, medicação e observações.
- Registro de preparação para treino: energia, sonolência, estado mental, dores musculares, alimentação pré-treino e observações.
- Salvamento automático dos campos no navegador, organizado por data.
- Recuperação de registros anteriores ao selecionar uma data.
- Geração de duas mensagens: uma para sono e outra para treino.
- Cópia para a área de transferência, com seleção manual da prévia quando a cópia automática falha.
- Exportação e importação de backups JSON.
- Remoção do registro da data selecionada mediante confirmação.

O MVP coleta e organiza dados. Ainda não calcula indicadores, apresenta gráficos, prescreve treinos ou interpreta a qualidade do sono.

## Como executar

Abra `sleep-to-train.html` em um navegador moderno. Não há instalação de pacotes nem etapa de compilação.

Para servir o arquivo localmente, caso tenha Python 3 instalado:

```bash
python3 -m http.server 8000 --bind 127.0.0.1
```

Acesse `http://localhost:8000/sleep-to-train.html`. Mantenha o mesmo navegador e endereço para acessar os registros armazenados naquela origem.

Abrir diretamente como arquivo (`file://`) pode apresentar limitações de persistência, especialmente no Android, e de acesso à área de transferência. A própria interface orienta exportar backups e usar cópia manual quando necessário.

## Uso diário

1. Confira a data do registro, inicialmente preenchida com a data local do dispositivo.
2. Preencha os dados de sono e de preparação para treino. Use `h:mm` para durações, por exemplo `7:22`.
3. Na seção **Meus chats no ChatGPT**, informe os endereços das conversas de sono e treino e clique em **Salvar links dos chats**. Use o endereço da barra do navegador, não um link compartilhado. Essa configuração é opcional e pode ser alterada quando o chat de treino mudar.
4. Clique em **Copiar e abrir Sono** ou **Copiar e abrir Treino**. Sem link configurado, os botões continuam apenas copiando.
5. Cole a mensagem no chat de acompanhamento desejado.
6. Use **Exportar backup JSON** para manter uma cópia dos registros fora do navegador.

Ao trocar a data, o formulário carrega o registro correspondente ou apresenta campos vazios. Cada data possui um único registro; editar seus campos atualiza esse registro.

**Limpar formulário** também remove o registro armazenado da data selecionada. Na importação, registros de datas presentes no backup substituem os registros locais das mesmas datas; as demais datas são preservadas.

## Tecnologias e estrutura

| Arquivo | Finalidade |
| --- | --- |
| `sleep-to-train.html` | Interface, estilos e toda a lógica do aplicativo |
| `health-checkin.html` | Redirecionamento de compatibilidade para links antigos |
| `index.html` | Entrada do site, que encaminha para o aplicativo |
| `README.md` | Visão geral e instruções de uso |
| `docs/analise-tecnica.md` | Arquitetura, dados, limitações e sugestões de evolução |

Tecnologias: HTML, CSS e JavaScript nativos, com `localStorage`, `FormData`, Clipboard API e APIs de arquivos do navegador. Não foram encontrados gerenciador de pacotes, configuração de build ou testes automatizados.

## Armazenamento e privacidade

Os registros são gravados como JSON na chave `health-checkin-v1` do `localStorage`. O código atual não faz requisições de rede nem envia dados automaticamente para serviços externos.

A chave interna mantém o nome original para preservar os dados existentes. Os novos backups usam o prefixo `sleep-to-train-backup-`; backups antigos continuam compatíveis.

Os dados ficam vinculados ao navegador e à origem em que a página foi aberta. Limpar os dados do navegador pode removê-los. Não há sincronização entre dispositivos, autenticação ou criptografia implementada pelo aplicativo. Backups incluem os campos preenchidos, inclusive medicação e observações. Ao colar uma mensagem em um chat, o compartilhamento é feito pelo próprio usuário.

Há uma falha conhecida no aviso de salvamento: a interface pode informar sucesso mesmo quando o navegador não consegue persistir os dados. Até a correção, mantenha backups e confira a recuperação dos registros após reabrir a página.

Consulte a [análise técnica inicial](docs/analise-tecnica.md) para os detalhes e próximos passos sugeridos.

## Publicação no GitHub Pages

1. Envie `index.html`, `sleep-to-train.html`, `health-checkin.html`, `.nojekyll`, `.gitignore`, `README.md` e `docs/` para um repositório no GitHub. Não inclua backups pessoais.
2. No repositório, abra **Settings → Pages**.
3. Em **Source**, selecione **Deploy from a branch**.
4. Selecione a branch que contém os arquivos (por exemplo, `main`), a pasta **/(root)** e clique em **Save**.
5. Aguarde a publicação e abra o endereço indicado pelo GitHub Pages.

O arquivo `index.html` encaminha para `./sleep-to-train.html`, preservando o funcionamento em endereços com o nome do repositório. O arquivo `.nojekyll` dispensa o processamento por Jekyll.

Ao migrar do arquivo local para o endereço publicado, os registros antigos não aparecem automaticamente: exporte o backup no endereço anterior e importe no site publicado. Cada navegador continua mantendo seus próprios dados; a publicação não adiciona sincronização.

Referência: [configuração da publicação no GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site).

Os links dos chats são armazenados separadamente na chave `sleep-to-train-chats-v1` e não fazem parte do backup dos registros. A cópia automática precisa da permissão do navegador; em caso de falha, a prévia é selecionada para cópia manual. Se a abertura da aba for bloqueada, use o link apresentado após a cópia. Nenhuma mensagem é enviada automaticamente ao ChatGPT.
