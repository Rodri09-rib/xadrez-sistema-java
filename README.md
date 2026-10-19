<div align="center">

# Xadrez

**Sistema de xadrez em console, em Java puro, com domínio de tabuleiro genérico**

[![Licença MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/Rodri09-rib/xadrez-sistema-java/blob/main/LICENSE)
[![Java 21](https://img.shields.io/badge/Java-21-orange.svg)](https://openjdk.org/)
[![Em construção](https://img.shields.io/badge/status-em%20construção-yelloworange.svg)](https://github.com/Rodri09-rib/xadrez-sistema-java)

</div>

## Sobre

O Xadrez é um jogo de xadrez em **100% Java puro**, com interface de console, sem
frameworks e sem dependências externas. O objetivo é construir o
jogo **de baixo para cima**, começando pelo que é genérico (um tabuleiro, uma posição,
uma peça) e só depois pelo que é específico do xadrez (as casas, as cores, as regras).

O ponto do projeto é a **separação entre os dois domínios**. O pacote `boardgame` não
sabe que existe xadrez: um tabuleiro com linhas, colunas e peças serviria para damas ou
para qualquer outro jogo. O pacote `chess` é que sabe: conhece a notação `a1`–`h8`, as
cores e, quando existirem, os movimentos de cada peça.

O projeto está organizado em **camadas por pacote** (`application`, `boardgame`,
`chess`, `chess.pieces`): `application` monta a partida, `chess` orquestra as regras,
`chess.pieces` define o comportamento de cada peça e `boardgame` fica em baixo, a servir
todos eles.

## Funcionalidades

**Tabuleiro**

- `Board` de 8 × 8 posições, com `placePiece`, `piece` e `thereIsAPiece`.
- Toda a validação passa por `BoardException`: posição fora do tabuleiro, posição já
  ocupada, tabuleiro criado com menos de uma linha ou uma coluna.

**Posicionamento inicial**

- `ChessMatch` cria o tabuleiro e já posiciona três peças: o rei branco em `e1`, o rei
  preto em `e8` e uma torre branca em `b6`.
- A notação de xadrez (`coluna` `a`–`h` + `linha` `1`–`8`) é convertida para as
  coordenadas internas (`0`–`7`, `0`–`7`) por `ChessPosition.toPosition()`.

**Impressão em console**

- `UI.printBoard` desenha o tabuleiro com as linhas numeradas de `8` a `1` e as colunas
  de `a` a `h`, como num tabuleiro real.
- Casa vazia aparece `-`; peça ocupada aparece pela letra do `toString()` (`K` para o rei,
  `R` para a torre).

## O que separa `boardgame` de `chess`

O projeto é o exercício de camadas, por isso vale apontar **onde** cada fronteira está.

**`boardgame` é genérico** `Board`, `Piece`, `Position` e `BoardException` não têm
nenhum import de `chess`. `Piece` tem uma `position` (inicialmente `null`) e um `board`
ao qual pertence; não tem cor, nem tipo, nem regra.

**`chess` especializa** `ChessPiece` estende `Piece` e acrescenta o que só existe no
xadrez: a `Color` (enum `BLACK`/`WHITE`). `King` e `Rook` estendem `ChessPiece`, e cada
um define a letra no `toString()`.

```java
Piece                  // boardgame: posição + tabuleiro
 └── ChessPiece        // chess: + cor
      ├── King         // chess.pieces: "K"
      └── Rook         // chess.pieces: "R"
```

**`ChessPosition` é a ponte** o construtor valida a notação (`a`–`h`, `1`–`8`, senão
`ChessException`), e `toPosition()` traduz para as coordenadas internas:
linha `8 - row`, coluna `column - 'a'`. Nenhuma outra classe do projeto
precisa saber que o xadrez se escreve com letras.

**Duas exceções, dois níveis** `BoardException` é do domínio genérico e é lançada pelo
`Board`; `ChessException` é do domínio do xadrez e é lançada por `ChessPosition`. As
duas são `RuntimeException`: são programação defensiva de invariantes, não condições
que o chamador tenha que antecipar no fluxo do programa.

## Decisões de projeto

**O `position` da `Piece` é `protected` e só o `boardgame` o escreve.** Quem coloca a
peça é `Board.placePiece`, que faz ao mesmo tempo `pieces[row][column] = piece` e
`piece.position = position`. As duas metades ficam juntas: uma peça nunca fica na matriz
sem saber onde está, nem saber onde está sem estar na matriz.

**`getPieces()` devolve uma cópia.** `ChessMatch.getPieces()` constrói uma matriz nova a
cada chamada e copia as peças. O chamador (neste caso a `UI`) recebe um retrato do
estado atual, não uma referência com que conseguisse mexer no tabuleiro por baixo da
partida.

**`ChessMatch` é o único dono do estado da partida.** A `UI` só lê: recebe a matriz e
imprime. Nenhuma peça conhece a partida, e a partida conhece as peças pelos nomes. É o
que vai permitir acrescentar turno, captura e xeque sem tocar na camada de impressão.

**Validação no construtor, não em setters.** `ChessPosition` recusa `i9` ou `a0` no
momento em que é criada; `Board` recusa `new Board(0, 8)`. Um objeto que sobrevive ao
construtor é um objeto válido, e não é preciso voltar a verificar mais tarde.

**`enum Color` em vez de `boolean isWhite`.** Dois valores nomeados leem-se no debug e
nas mensagens de erro; um boolean obriga a decorar o que `true` quer dizer naquela linha.

## Stack

| Camada | Tecnologias |
| --- | --- |
| Linguagem | Java 21 (sem framework, sem dependências) |
| Interface | Console, `System.out` e menu em `UI` |
| Domínio genérico | `boardgame`: `Board`, `Piece`, `Position`, `BoardException` |
| Domínio do xadrez | `chess`: `ChessMatch`, `ChessPiece`, `ChessPosition`, `Color` |
| Peças | `chess.pieces`: `King`, `Rook` (Rainha, Bispo, Cavalo e Peão por criar) |
| Exceções | `BoardException` e `ChessException`, ambas unchecked |
| Build | Módulo IntelliJ (`xadrez-sistema.iml`); sem Maven nem Gradle por enquanto |

Sem dependências de runtime nem de teste: o `src` inteiro compila com o `javac` sozinho.

## Como executar

**Requisitos:** JDK 21+. Nada mais: não há base de dados nem variáveis de ambiente a
carregar.

Pela IDE: abra `application.Program` e execute o `main`.

Pela linha de comandos:

```bash
javac -d out $(find src -name "*.java")   # compila todos os pacotes para out/
java -cp out application.Program          # imprime o tabuleiro com as 3 peças
```

A saída é o tabuleiro 8 × 8 com as três peças do `initialSetup`:

```
8 - - - - K - - -
7 - - - - - - - -
6 - R - - - - - -
5 - - - - - - - -
4 - - - - - - - -
3 - - - - - - - -
2 - - - - - - - -
1 - - - - K - - -
  a b c d e f g h
```

## Estrutura

```
src/
├── application/
│   ├── Program.java            main: cria a ChessMatch e imprime o tabuleiro
│   └── UI.java                 impressão do tabuleiro (linha 8→1, coluna a→h)
├── boardgame/                  domínio genérico, sem import de chess
│   ├── Board.java              matriz de peças + validação de posição
│   ├── Piece.java              peça genérica: posição (null até ser colocada) + tabuleiro
│   ├── Position.java           coordenada interna (row, column), 0-based
│   └── BoardException.java     erro do domínio genérico (unchecked)
├── chess/                      regras e objetos do xadrez
│   ├── ChessMatch.java         a partida: tabuleiro, posicionamento inicial, estado
│   ├── ChessPiece.java         Piece + cor
│   ├── ChessPosition.java      notação a1–h8 e conversão para Position
│   ├── Color.java              enum BLACK, WHITE
│   └── ChessException.java     erro do domínio do xadrez (unchecked)
└── chess/pieces/
    ├── King.java               "K"
    └── Rook.java               "R"
```

O sentido das dependências é de cima para baixo e sempre num só sentido:
`application` conhece `chess`; `chess` conhece `boardgame`; `boardgame` não conhece
ninguém. Uma peça não conhece a partida que a usa.

### Mapa de responsabilidades

| Camada | Responde por | Não faz |
| --- | --- | --- |
| `application` | Criar a partida, imprimir o tabuleiro | Regras de xadrez |
| `chess` | Estado da partida, notação, cores | Desenhar no console |
| `chess.pieces` | A letra de cada peça no tabuleiro | Saber onde estão as outras |
| `boardgame` | Matriz, posições, invariantes do tabuleiro | Saber que o jogo é xadrez |

## Limitações conhecidas

São limitações de um projeto em construção, e não esquecimentos. O repositório tem 8
commits e o trabalho continua:

- **Ainda não há movimento.** Nenhuma peça tem método de movimento: `King` e `Rook` só
  têm cor e `toString()`. `ChessMatch` posiciona as peças e nada mais.
- **Não há turno, captura, xeque nem xeque-mate.** O coração do jogo está por construir:
  `possibleMoves`, `makeMove` e as validações de xeque.
- **Só existem o Rei e a Torre.** Rainha, Bispo, Cavalo e Peão não estão criados.
- **O console não lê jogadas.** `Program` cria a partida e imprime; não há `Scanner` nem
  menu.
- **`ChessPosition.fromPosition` está errada e não é usada.** Subtrai em vez de somar a
  coluna (`'a' - column` em vez de `'a' + column`); vai ser necessária quando existir
  notação de saída e é das primeiras coisas a corrigir.
- **`UI` assume tabuleiro quadrado.** O laço das colunas usa `pieces.length` (linhas) em
  vez de `pieces[0].length`; num tabuleiro 8 × 8 não se nota.
- **Sem testes e sem build tool.** O projeto é um módulo IntelliJ, sem `pom.xml`, sem
  JUnit e sem Maven: a suíte de testes entra quando a lógica de movimento justificar.
- **Não há licença no repositório.** O projeto declara MIT no perfil, mas o arquivo
  `LICENSE` ainda não foi criado.

## Autor

**Rodrigo Ribeiro Ferreira**  [LinkedIn](https://www.linkedin.com/in/rodrigo-ribeiro-abbb713aa?utm_source=share_via&utm_content=profile&utm_medium=member_ios)

## Licença

[MIT](LICENSE)
