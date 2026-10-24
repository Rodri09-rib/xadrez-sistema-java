<div align="center">

# Xadrez

**Sistema de xadrez em console, em Java puro, com domínio de tabuleiro genérico**

[![Licença MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/Rodri09-rib/xadrez-sistema-java/blob/main/LICENSE)
[![Java 21](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/)
[![Concluído](https://img.shields.io/badge/status-concluído-brightgreen.svg)](https://github.com/Rodri09-rib/xadrez-sistema-java)

</div>

## Sobre

O Xadrez é um jogo de xadrez em **100% Java puro**, com interface de console, sem frameworks e sem dependências externas. O objetivo é construir o jogo **de baixo para cima**, começando pelo que é genérico (um tabuleiro, uma posição, uma peça) e só depois pelo que é específico do xadrez (as casas, as cores, as regras).

O ponto do projeto é a **separação entre os dois domínios**. O pacote `boardgame` não sabe que existe xadrez: um tabuleiro com linhas, colunas e peças serviria para damas ou para qualquer outro jogo. O pacote `chess` é que sabe: conhece a notação `a1`–`h8`, as cores e as regras do xadrez.

O projeto está organizado em **camadas por pacote** (`application`, `boardgame`, `chess`, `chess.pieces`): `application` monta a partida e controla a interface, `chess` orquestra as regras, `chess.pieces` define o comportamento de cada peça e `boardgame` fica em baixo, a servir todos eles.

## Funcionalidades

### Tabuleiro

- `Board` de 8 × 8 posições, com `placePiece`, `piece`, `removePiece` e `thereIsAPiece`.
- Toda a validação passa por `BoardException`: posição fora do tabuleiro, posição já ocupada, tabuleiro criado com menos de uma linha ou uma coluna.

### Regras completas do xadrez

- **Posicionamento inicial**: todas as 32 peças são posicionadas corretamente na partida.
- **Movimentação por peça**: Rei, Dama, Torre, Bispo, Cavalo e Peão com seus movimentos específicos.
- **Captura**: Peças podem capturar adversárias, removendo-as do tabuleiro.
- **Xeque**: Detecção de xeque ao rei.
- **Xeque-mate**: Detecção de xeque-mate e fim de partida.
- **Roque**: Roque pequeno e grande (com todas as validações).
- **En passant**: Captura especial do peão.
- **Promoção**: Peão pode ser promovido ao chegar à última fileira.

### Interface

- **Notação algébrica**: Conversão entre notação `a1`–`h8` e coordenadas internas.
- **Impressão em console**: Tabuleiro desenhado com linhas numeradas de `8` a `1` e colunas de `a` a `h`. Casa vazia aparece `-`, peça ocupada aparece pela letra (`K`, `Q`, `R`, `B`, `N`, `P`).
- **Sistema de turnos**: Alternância entre brancas e pretas.
- **Jogadas possíveis**: Visualização das casas válidas para movimento.
- **Tratamento de erros**: Exceções claras para jogadas inválidas.

## O que separa `boardgame` de `chess`

O projeto é o exercício de camadas, por isso vale apontar **onde** cada fronteira está.

**`boardgame` é genérico** `Board`, `Piece`, `Position` e `BoardException` não têm nenhum import de `chess`. `Piece` tem uma `position` (inicialmente `null`) e um `board` ao qual pertence; não tem cor, nem tipo, nem regra.

**`chess` especializa** `ChessPiece` estende `Piece` e acrescenta o que só existe no xadrez: a `Color` (enum `BLACK`/`WHITE`). Todas as peças (`King`, `Queen`, `Rook`, `Bishop`, `Knight`, `Pawn`) estendem `ChessPiece`, e cada uma define a letra no `toString()` e seu próprio `possibleMoves()`.

```java
Piece                  // boardgame: posição + tabuleiro
 └── ChessPiece        // chess: + cor
      ├── King         // chess.pieces: "K"
      ├── Queen        // chess.pieces: "Q"
      ├── Rook         // chess.pieces: "R"
      ├── Bishop       // chess.pieces: "B"
      ├── Knight       // chess.pieces: "N"
      └── Pawn         // chess.pieces: "P"
```

**`ChessPosition` é a ponte** o construtor valida a notação (`a`–`h`, `1`–`8`, senão `ChessException`), e `toPosition()` traduz para as coordenadas internas: linha `8 - row`, coluna `column - 'a'`. Nenhuma outra classe do projeto precisa saber que o xadrez se escreve com letras.

**Duas exceções, dois níveis** `BoardException` é do domínio genérico e é lançada pelo `Board`; `ChessException` é do domínio do xadrez e é lançada por `ChessPosition`. As duas são `RuntimeException`: são programação defensiva de invariantes, não condições que o chamador tenha que antecipar no fluxo do programa.

## Decisões de projeto

**O `position` da `Piece` é `protected` e só o `boardgame` o escreve.** Quem coloca a peça é `Board.placePiece`, que faz ao mesmo tempo `pieces[row][column] = piece` e `piece.position = position`. As duas metades ficam juntas: uma peça nunca fica na matriz sem saber onde está, nem saber onde está sem estar na matriz.

**`getPieces()` devolve uma cópia.** `ChessMatch.getPieces()` constrói uma matriz nova a cada chamada e copia as peças. O chamador (neste caso a `UI`) recebe um retrato do estado atual, não uma referência com que conseguisse mexer no tabuleiro por baixo da partida.

**`ChessMatch` é o único dono do estado da partida.** A `UI` só lê: recebe a matriz e imprime. Nenhuma peça conhece a partida, e a partida conhece as peças pelos nomes. Isso permite gerenciar turno, xeque, xeque-mate e regras especiais sem acoplar regras às peças ou à interface.

**Validação no construtor, não em setters.** `ChessPosition` recusa `i9` ou `a0` no momento em que é criada; `Board` recusa `new Board(0, 8)`. Um objeto que sobrevive ao construtor é um objeto válido, e não é preciso voltar a verificar mais tarde.

**`enum Color` em vez de `boolean isWhite`.** Dois valores nomeados leem-se no debug e nas mensagens de erro; um boolean obriga a decorar o que `true` quer dizer naquela linha.

## Stack

| Camada | Tecnologias |
| --- | --- |
| Linguagem | Java 21 (sem framework, sem dependências) |
| Interface | Console, `System.out` e menu em `UI` |
| Domínio genérico | `boardgame`: `Board`, `Piece`, `Position`, `BoardException` |
| Domínio do xadrez | `chess`: `ChessMatch`, `ChessPiece`, `ChessPosition`, `Color` |
| Peças | `chess.pieces`: `King`, `Queen`, `Rook`, `Bishop`, `Knight`, `Pawn` |
| Exceções | `BoardException` e `ChessException`, ambas unchecked |
| Build | Módulo IntelliJ (`xadrez-sistema.iml`); sem Maven nem Gradle por enquanto |

Sem dependências de runtime nem de teste: o `src` inteiro compila com o `javac` sozinho.

## Como executar

**Requisitos:** JDK 21+. Nada mais: não há base de dados nem variáveis de ambiente a carregar.

Pela IDE: abra `application.Program` e execute o `main`.

Pela linha de comandos:

```bash
javac -d out $(find src -name "*.java")   # compila todos os pacotes para out/
java -cp out application.Program          # inicia o jogo de xadrez
```

Ao iniciar, o programa exibe o tabuleiro completo com todas as peças posicionadas, solicita a jogada no formato `origem destino` (ex.: `e2 e4`) e exibe as casas possíveis ao selecionar uma peça.

## Estrutura

```
src/
├── application/
│   ├── Program.java            main: cria a ChessMatch e inicia o jogo
│   └── UI.java                 impressão do tabuleiro, leitura e exibição de jogadas
├── boardgame/                  domínio genérico, sem import de chess
│   ├── Board.java              matriz de peças + validação de posição
│   ├── Piece.java              peça genérica: posição (null até ser colocada) + tabuleiro
│   ├── Position.java           coordenada interna (row, column), 0-based
│   └── BoardException.java     erro do domínio genérico (unchecked)
├── chess/                      regras e objetos do xadrez
│   ├── ChessMatch.java         partida: tabuleiro, turnos, xeque, xeque-mate, regras especiais
│   ├── ChessPiece.java         Piece + cor
│   ├── ChessPosition.java      notação a1–h8 e conversão para Position
│   ├── Color.java              enum BLACK, WHITE
│   └── ChessException.java     erro do domínio do xadrez (unchecked)
└── chess/pieces/
    ├── King.java               Rei
    ├── Queen.java              Dama
    ├── Rook.java               Torre
    ├── Bishop.java             Bispo
    ├── Knight.java             Cavalo
    └── Pawn.java               Peão
```

O sentido das dependências é de cima para baixo e sempre num só sentido:
`application` conhece `chess`; `chess` conhece `boardgame`; `boardgame` não conhece
ninguém. Uma peça não conhece a partida que a usa.

### Mapa de responsabilidades

| Camada | Responde por | Não faz |
| --- | --- | --- |
| `application` | Criar a partida, imprimir o tabuleiro, ler jogadas | Regras de xadrez |
| `chess` | Estado da partida, turnos, xeque, xeque-mate, roque, en passant, promoção | Desenhar no console |
| `chess.pieces` | Movimentação de cada peça, `toString()` | Saber onde estão as outras peças |
| `boardgame` | Matriz, posições, invariantes do tabuleiro | Saber que o jogo é xadrez |

## Limitações conhecidas

- **Sem testes automatizados.** O projeto é um módulo IntelliJ, sem `pom.xml`, sem JUnit. A lógica de movimentação está implementada, mas não há suíte de testes.
- **Sem persistência de partidas.** Apenas sessão em memória, sem salvar/carregar jogos.
- **Sem interface gráfica.** Execução exclusivamente em console.

## Autor

**Rodrigo Ribeiro Ferreira** - [LinkedIn](https://www.linkedin.com/in/rodrigo-ribeiro-abbb713aa?utm_source=share_via&utm_content=profile&utm_medium=member_ios) | [GitHub](https://github.com/Rodri09-rib)

## Licença

[MIT](LICENSE)
