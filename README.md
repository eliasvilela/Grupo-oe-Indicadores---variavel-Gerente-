# Remuneração Variável — Gerente | Grupo OE

Painel de apuração mensal da remuneração variável do Gerente, desenvolvido pela **PWR Gestão** para o Grupo OE (Yoto, Ponza e Cortile).

Substitui a planilha `RV grupo oe.xlsx` por uma ferramenta que o gerente preenche e o diretor audita, com histórico mês a mês e parâmetros ajustáveis sem mexer no código.

---

## As três abas

**Apuração** — o gerente lança o resultado de cada indicador. A pontuação e a comissão se atualizam a cada lançamento, com barra de realizado contra a meta.

**Resultado** — leitura do fechamento: aproveitamento por grupo de indicadores, tabela de meta contra resultado com desvio percentual, lista dos pontos perdidos convertidos em reais e histórico de comissão mês a mês. Daqui saem a exportação em Excel e o PDF.

**Parâmetros** — o diretor ajusta meta, peso, sentido da comparação, forma de apuração, grupo e teto da comissão. Também inclui e remove indicadores.

---

## Modelo de cálculo

```
valor do ponto = teto da comissão ÷ pontos ativos
comissão       = pontos alcançados × valor do ponto
```

- Indicador só pontua quando o resultado é lançado. Campo em branco vale zero.
- Apuração por indicador: **tudo ou nada** (padrão) ou **proporcional**, limitada a 100% da meta.
- Sentido da comparação por indicador: maior ou igual, ou menor ou igual.
- Indicador por unidade: **soma** trata branco como zero; **média** considera apenas as unidades lançadas e sinaliza lançamento incompleto.
- Mês fechado congela os parâmetros vigentes no fechamento. Reabrir volta a usar os vigentes.

### Indicadores do modelo

| Grupo | Indicador | Sentido | Pontos |
|---|---|---|---|
| Faturamento | Yoto, Ponza e Cortile | ≥ | 1 cada |
| Avaliações de clientes | Nota média Tripadvisor (média das unidades) | ≥ | 1 |
| Avaliações de clientes | Google 5 estrelas (soma das unidades) | ≥ | 1 |
| Avaliações de clientes | Risposta | ≥ | 1 |
| Check lists | Check list diário do gerente | ≥ | 5 |
| Check lists | Check list diário da equipe | ≥ | 5 |
| Consumo | Água, energia e gás | ≤ | 1 |
| Extras | Dobras | ≤ | 1 |
| Material descartável e limpeza | Tolerância em dias com item em falta | ≤ | 1 |
| Material descartável e limpeza | Gasto com material de limpeza | ≤ | 1 |

Total de 20 pontos no modelo padrão.

> **Os valores de meta neste repositório são ilustrativos.** As metas reais, o teto e os pesos de cada competência são definidos na aba Parâmetros e ficam no banco privado do painel publicado. Nada de confidencial do cliente está neste código.

---

## Divergências encontradas na planilha original

Levantadas na migração e tratadas no painel:

1. `G17` somava apenas `G8:G16`, deixando os três pontos de faturamento fora da comissão. Corrigido: os 20 pontos entram na apuração.
2. Tripadvisor dividia a soma das três notas por 5 em vez de 3, gerando meta 2,9 em vez de 4,83. Corrigido para média das unidades.
3. Risposta comparava a média das notas do Google contra uma meta de 100. Fórmula incoerente; virou lançamento direto em percentual. **Definição do indicador pendente com o cliente.**
4. As colunas de nota do Google por unidade existiam na planilha mas nenhum indicador pontuado as usava. Não foram incluídas; podem ser adicionadas na aba Parâmetros.

---

## Acesso

Esta cópia abre com tela de login. Credencial inicial:

```
usuário: grupooe
senha:   Gerente@2026
```

Para trocar, gere o hash da nova credencial e substitua `HASH_ACESSO` no `index.html`:

```bash
node -e "console.log(require('crypto').createHash('sha256').update('usuario:senha').digest('hex'))"
```

O usuário não diferencia maiúscula de minúscula; a senha diferencia. A sessão vale enquanto a aba estiver aberta.

> **O que essa tela é e o que não é.** É um porteiro, não um cofre. O arquivo roda inteiro no navegador, então quem souber abrir o código-fonte contorna a tela. Serve para impedir acesso casual de quem receber o link. Proteção de verdade exige servidor com autenticação, ou o painel publicado no claude.ai, onde o controle de acesso é do próprio compartilhamento.

---

## Como rodar

O arquivo é autocontido. Abrir `index.html` no navegador já funciona, com os dados guardados no próprio navegador (`localStorage`).

### Onde ficam os dados

| Como você abre | Onde os lançamentos ficam | Compartilha entre pessoas |
|---|---|---|
| `index.html` no navegador, ou hospedado (GitHub Pages, servidor) | No navegador de cada um (`localStorage`) | Não |
| Painel publicado no claude.ai | Banco do artifact, na nuvem | Sim, diretor e gerente veem o mesmo |

**Não é preciso montar banco de dados** para a versão publicada: o armazenamento já vem com o artifact. A cópia deste repositório serve como código-fonte, histórico de versões e alternativa de uso individual. Se um dia o painel sair do claude.ai e precisar ser compartilhado entre pessoas, aí sim será necessário um backend.

A versão publicada roda como artifact no claude.ai, onde ganha armazenamento compartilhado entre diretor e gerente e controle de acesso:

- **Diretor** — acesso "Pode editar": altera parâmetros e lança resultados.
- **Gerente** — acesso "Pode interagir": lança resultados e consulta os parâmetros em leitura.

Quem abre sem esse ajuste cai em "Pode interagir" e a aba Parâmetros fica em modo leitura.

---

## Notas técnicas

O painel roda em ambientes onde os eventos de digitação e foco do navegador não chegam à página, o que exigiu decisões fora do comum:

- **Objetos do servidor chegam congelados.** Tudo que vem do armazenamento compartilhado é clonado antes de entrar no estado; escrever direto no objeto recebido lança exceção.
- **Foco detectado por sondagem**, a cada 400 ms, via `document.activeElement`. Não depende de evento de foco.
- **Leitura restrita ao campo em uso.** Campo parado nunca empurra valor para o cálculo, o que evita o laço em que uma varredura ressuscitava dado apagado.
- **Escrita no sentido inverso.** Todo campo que a pessoa não está usando é alinhado ao estado no mesmo intervalo, o que descarta valores repostos pela plataforma.
- **Confirmações em dois toques**, dentro da página. A janela nativa do navegador não responde dentro do aplicativo.
- **Exportações carregadas sob demanda**: SheetJS para o Excel e jsPDF para o PDF, só no clique.

### Validação

13 baterias automatizadas, 287 verificações, mais 250 cenários aleatórios conferidos contra uma implementação independente da regra de cálculo (1.500 comparações). Cobrem cálculo, parâmetros, inclusão e remoção de indicadores, armazenamento compartilhado, permissões, casos-limite, objetos congelados, eventos desligados e reposição de formulário pela plataforma.

---

## Identidade visual

Painel na marca do cliente: logo do Grupo OE no cabeçalho azul-marinho, PWR assinando no rodapé. Paleta restrita a azul-marinho, laranja e branco. Tema claro e escuro.

---

<sub>Desenvolvido por **PWR Gestão** · pwrgestao.com</sub>
