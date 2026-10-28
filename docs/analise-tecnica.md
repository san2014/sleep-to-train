# Análise técnica inicial

Análise estática realizada em 27/10/2026. O arquivo do aplicativo foi posteriormente renomeado para `sleep-to-train.html`; `health-checkin.html` passou a redirecionar para ele. Este documento descreve o código disponível e não representa validação de execução em navegadores ou dispositivos reais.

## Objetivo e escopo

O projeto reduz o trabalho de transcrever dados de sono e preparação para treino em chats de acompanhamento. O usuário fornece os dados, o navegador mantém registros por data e o aplicativo monta mensagens padronizadas.

O rótulo da interface é `V1`; o backup utiliza `version: 1`. Não há histórico de versões disponível no material analisado.

## Arquitetura

Toda a implementação está em um documento: CSS no `<style>`, interface no `<body>` e JavaScript no `<script>`. O processamento ocorre no navegador.

Fluxo principal:

```text
Abertura → leitura do localStorage → formulário do dia → prévia
                                         ↓
Edição → coleta dos campos → registro por data → localStorage
                          └→ atualização da prévia

Botão de cópia → seleção do tipo de mensagem → área de transferência
Exportação → registros em memória → arquivo JSON
Importação → leitura do JSON → confirmação → mesclagem por data
```

| Elemento | Responsabilidade |
| --- | --- |
| `safeGet` / `safeSet` | Encapsular acesso ao armazenamento e capturar falhas |
| `today` | Produzir a data local em `YYYY-MM-DD` |
| `fields` / `fill` | Ler e preencher o formulário |
| `records` | Manter os registros em memória, indexados pela data |
| `persist` | Atualizar o registro selecionado e tentar persistir o conjunto |
| `sleepText` / `trainingText` | Montar as mensagens, omitindo campos vazios |
| `render` / `mode` | Atualizar a prévia conforme o modo atual |
| `maskDuration` | Aplicar formatação parcial às durações digitadas |
| `copy` | Tentar copiar e selecionar a prévia em caso de falha |

Na inicialização, o aplicativo carrega o registro do dia atual e mostra a mensagem de sono. A escolha de mensagem não é persistida. Alterações de campos atualizam a prévia; a troca de data recupera os valores daquela data sem criar imediatamente um novo registro.

## Modelo de dados

Os valores obtidos por `FormData` são strings, inclusive números. Campos vazios são armazenados como `""`. Não há identificador de usuário, horário de criação ou horário de alteração.

| Campo | Conteúdo | Restrições declaradas no HTML |
| --- | --- | --- |
| `date` | Data do registro e chave em `records` | Data, obrigatória |
| `wake` | Horário de despertar | Horário |
| `total`, `deep`, `rem`, `light` | Durações de sono | Padrão `h:mm` ou `hh:mm`, minutos de 00 a 59, até 5 caracteres |
| `awakenings` | Quantidade de despertares | Número de 0 a 99 |
| `awake` | Tempo acordado em minutos | Número de 0 a 1440 |
| `quality` | Qualidade percebida | Número de 0 a 10 |
| `medication` | Nome, dose e horário informados pelo usuário | Texto livre |
| `sleepNotes` | Observações sobre o sono | Texto livre |
| `energy`, `sleepiness` | Energia e sonolência | Número de 0 a 10 |
| `mental` | Estado mental | Vazio, Focado, Sonolento, Irritado, Cansado ou Disposto |
| `pain` | Dores musculares | Texto livre |
| `meal` | Alimentação pré-treino | Texto livre |
| `trainingNotes` | Observações do treino | Texto livre |

As restrições HTML não constituem validação efetiva do fluxo: os botões não submetem o formulário e o código não chama `checkValidity()` ou `reportValidity()` antes de salvar, copiar ou exportar.

### Persistência e backup

No `localStorage`, a chave `health-checkin-v1` contém diretamente o objeto de registros. No arquivo de backup, esse objeto recebe um envelope com versão:

```json
{
  "version": 1,
  "records": {
    "2026-10-27": {
      "date": "2026-10-27",
      "wake": "06:30",
      "total": "7:22",
      "deep": "1:34",
      "rem": "2:01",
      "light": "3:47",
      "awakenings": "0",
      "awake": "0",
      "quality": "8",
      "medication": "",
      "sleepNotes": "",
      "energy": "8",
      "sleepiness": "2",
      "mental": "Disposto",
      "pain": "Nenhuma",
      "meal": "Banana e aveia",
      "trainingNotes": ""
    }
  }
}
```

O exemplo é fictício. A exportação tenta salvar os campos atuais antes de gerar `sleep-to-train-backup-YYYY-MM-DD.json`, incluindo todos os registros em memória. A data no nome do arquivo é a data local da exportação.

A importação aceita apenas `version === 1` e exige `records` como objeto não nulo e não array. Após confirmação, mescla os registros com precedência do arquivo e volta a exibir o dia atual. Não valida cada registro, suas datas ou seus campos.

## Conteúdo das mensagens

As duas mensagens incluem data, horário de despertar, durações dos estágios, despertares e tempo acordado, quando preenchidos. Valores zero são preservados.

- **Sono:** acrescenta qualidade percebida, medicação e observações de sono.
- **Treino:** acrescenta energia, sonolência, estado mental, dores, alimentação e observações de treino.

A mensagem de treino não inclui qualidade percebida, medicação ou observações de sono. Os botões atualizam a prévia e tentam copiar; não existe envio direto para chats.

## Pontos positivos

- Implementação pequena, sem configuração de infraestrutura ou dependências externas.
- Separação das mensagens conforme o uso, evitando transcrição repetida.
- Registros por data e backup portável em JSON.
- Layout responsivo, rótulos de campos, foco visível e região de status com `aria-live`.
- Tratamento alternativo quando a cópia automática está indisponível.

## Limitações encontradas

| Prioridade sugerida | Achado no código | Consequência |
| --- | --- | --- |
| Alta | `safeSet` retorna `false` em caso de falha, mas `persist` ignora o retorno e anuncia sucesso | O usuário pode acreditar que há persistência quando os dados estão apenas em memória |
| Alta | Importação e limpeza também ignoram o retorno de `safeSet` | O estado após recarregar pode divergir da operação informada |
| Alta | Importação verifica apenas o envelope do backup | Registros incompatíveis podem ser incorporados e prejudicar o preenchimento ou a geração de mensagens |
| Média | Salvamento, cópia e exportação não verificam a validade dos campos | Datas vazias, durações incompletas e valores fora das faixas podem ser armazenados ou compartilhados |
| Média | Não há conferência entre sono total e soma dos estágios | Dados inconsistentes podem passar despercebidos |
| Média | Erros de leitura ou JSON inválido no armazenamento são tratados silenciosamente | Os registros podem parecer ausentes sem explicação ao usuário |
| Média | Não há lista de datas registradas ou busca de histórico | Para recuperar um registro, é necessário selecionar sua data |
| Baixa | HTML, CSS e JavaScript estão concentrados e parcialmente compactados | A manutenção tende a ficar mais difícil conforme o projeto cresce |

Não há gráficos, agregações semanais, integração com dispositivos, sincronização, service worker ou instalação como PWA. O funcionamento local não equivale a suporte a PWA. Também não foram encontrados testes automatizados.

## Evolução sugerida

As propostas abaixo não estão implementadas e não representam requisitos já aprovados.

1. **Confiabilidade dos dados:** respeitar o retorno de `safeSet`, informar falhas de leitura e validar registros importados antes de alterar o estado atual.
2. **Validação de entrada:** verificar datas, limites e formato de duração antes de concluir operações. Definir como tratar preenchimentos parciais durante o salvamento automático.
3. **Histórico acessível:** listar datas com registros e indicar claramente o dia em edição.
4. **Indicadores descritivos:** apresentar totais e médias de duração do sono, energia e sonolência, com período e quantidade de registros explícitos.
5. **Manutenção:** separar arquivos quando o crescimento justificar e adicionar testes para persistência, importação e mensagens, incluindo campos vazios e valores zero.

Antes de adicionar correlações entre sono e treino, definir quais resultados do treino serão coletados. Hoje o formulário registra principalmente o estado antes do treino; não há campos estruturados de carga, volume ou desempenho posterior.

## Verificação manual recomendada

Este roteiro fica como referência; não foi executado nesta análise.

- Preencher um dia, recarregar a página e verificar a recuperação dos campos.
- Alternar entre duas datas e conferir a independência dos registros.
- Testar mensagens de sono e treino com campos vazios e valores zero.
- Exportar e reimportar um backup, verificando substituição na mesma data e preservação das outras datas.
- Conferir tratamento de JSON inválido, versão incompatível e estrutura de registro inválida.
- Limpar uma data, recarregar e conferir que outra data permanece disponível.
- Testar armazenamento bloqueado e cópia indisponível, conferindo os avisos.
- Conferir o layout e a interação no navegador móvel utilizado no dia a dia.
