# Arquitetura do Computador de 8 Bits

**Status do documento:** Em desenvolvimento
**Versão:** 0.1
**Repositório:** `8bits-computer`
**Nome do computador:** Ainda não definido

---

## 1. Objetivo

Este documento registra a arquitetura definida para o computador de 8 bits desenvolvido pelo grupo.

O projeto tem como objetivo construir, primeiro em **simulação no Logisim**, e posteriormente em **hardware físico utilizando lógica digital discreta**, um computador simples capaz de executar instruções armazenadas em memória.

O documento deve funcionar como a **fonte de verdade da arquitetura**. Decisões posteriores que alterem qualquer característica estrutural do computador devem ser registradas aqui e, quando apropriado, também no histórico de decisões em `docs/decisions/decisions.md`.

A implementação física deverá ser derivada da arquitetura validada em simulação, evitando que decisões de hardware sejam tomadas prematuramente.

---

# 2. Status das decisões

Para evitar confusão entre o que já foi decidido e o que ainda está sendo projetado, este documento utiliza três categorias:

### Confirmado

Decisão já estabelecida pelo grupo e que deve ser tratada como requisito da arquitetura atual.

### Provisório

Decisão ou direção de projeto já considerada pelo grupo, mas que ainda depende de validação, detalhamento ou eventual revisão durante a simulação.

### Em aberto

Questão que ainda não foi decidida ou cujo detalhe necessário para implementação ainda não foi definido.

---

# 3. Arquitetura confirmada

## 3.1 Largura de dados

**Confirmado:** o computador possui **palavra de 8 bits**.

Consequentemente:

* registradores de dados possuem, em princípio, 8 bits;
* o barramento principal de dados possui 8 bits;
* a ALU opera sobre valores de 8 bits;
* a memória armazena palavras de 8 bits.

---

## 3.2 Espaço de endereçamento

**Confirmado:** o computador utiliza **endereços de 4 bits**.

Isso produz:

$$
2^4 = 16
$$

posições endereçáveis.

Portanto, a memória de programa/dados prevista possui:

* **16 endereços**
* **8 bits por endereço**
* capacidade total de:

$$
16 \times 8 = 128\text{ bits} = 16\text{ bytes}
$$

A faixa de endereços é:

```text
0000 → 0000
0001 → 0001
...
1110 → 1110
1111 → 1111
```

ou, em representação decimal:

```text
0 → 15
```

---

## 3.3 Formato das instruções

**Confirmado:** todas as instruções possuem **1 byte**, ou seja, 8 bits.

O formato é:

```text
┌────────────┬────────────┐
│   OPCODE   │  OPERANDO  │
│   4 bits   │   4 bits   │
└────────────┴────────────┘
```

Assim:

* **Opcode:** 4 bits
* **Operando/endereço:** 4 bits

O opcode permite representar até:

$$
2^4 = 16
$$

operações distintas.

O campo de operando pode representar diretamente qualquer um dos 16 endereços da memória.

---

## 3.4 Ciclo de máquina

**Confirmado:** o computador utiliza um ciclo de máquina fixo composto por **6 estados temporais**:

```text
T0 → T1 → T2 → T3 → T4 → T5 → T0 → ...
```

Cada instrução é executada por meio desses estados temporais.

O significado exato de cada estado será definido durante o projeto das micro-operações, mas a arquitetura deverá ser compatível com um ciclo de seis etapas.

---

## 3.5 Unidade de controle

**Confirmado:** a unidade de controle será **hardwired**.

Isso significa que os sinais de controle serão produzidos por lógica combinacional e sequencial dedicada, em vez de uma memória de microprograma.

A implementação conceitual será baseada em:

* decodificação do opcode;
* decodificação do estado temporal `T0–T5`;
* combinação desses sinais através de lógica digital;
* geração dos sinais necessários para controlar os módulos do processador.

A lógica exata da unidade de controle ainda precisa ser derivada a partir das micro-operações de cada instrução.

---

## 3.6 ALU

**Confirmado:** a ALU deverá executar as seguintes operações:

```text
ADD
SUB
AND
OR
```

Todas as operações trabalham com operandos de 8 bits.

### ADD

Adição de dois valores de 8 bits:

$$
A + B
$$

### SUB

Subtração de dois valores de 8 bits:

$$
A - B
$$

A implementação física esperada deverá utilizar a representação por complemento de dois, de forma equivalente a:

$$
A-B = A+\overline{B}+1
$$

O detalhamento da implementação ainda será validado na fase de simulação.

### AND

Operação AND bit a bit:

$$
A \land B
$$

### OR

Operação OR bit a bit:

$$
A \lor B
$$

---

## 3.7 Flags

**Confirmado:** o processador possui pelo menos duas flags:

* **Zero (`Z`)**
* **Carry (`C`)**

As flags serão armazenadas em **flip-flops D**, preservando seu estado entre os ciclos necessários.

### Zero

A flag `Z` indica que o resultado de uma operação é igual a zero.

Conceitualmente:

$$
Z =
\begin{cases}
1 & \text{se resultado}=0\\
0 & \text{caso contrário}
\end{cases}
$$

### Carry

A flag `C` registra o carry produzido pela operação aritmética quando aplicável.

O comportamento exato de `C` para cada operação da ALU, especialmente `SUB`, ainda deverá ser formalizado.

---

## 3.8 Memória

**Confirmado:** o sistema utiliza memória de **16 × 8 bits**.

Isto é:

```text
16 posições
8 bits por posição
```

A memória deverá armazenar instruções e dados, conforme definido pela ISA.

A programação da memória será realizada manualmente por meio de chaves/DIP switches.

---

## 3.9 Programação manual da memória

**Confirmado:** haverá um modo destinado à **programação manual da memória**.

Nesse modo, o usuário deverá conseguir selecionar:

* um endereço de 4 bits;
* um valor de dados de 8 bits;

e gravar esse valor na posição correspondente da memória.

Também haverá um modo normal de execução do computador.

A seleção entre os modos de programação e execução faz parte da arquitetura geral do sistema.

---

## 3.10 Modos de operação

**Confirmado:** o computador possuirá pelo menos os seguintes modos:

### PROG

Utilizado para programação manual da memória.

### RUN

Utilizado para execução normal do programa armazenado.

A seleção entre esses modos será realizada por hardware.

---

# 4. Arquitetura funcional em alto nível

A arquitetura funcional prevista pode ser representada de forma simplificada como:

```text
                 ┌─────────────────────┐
                 │      CONTROL        │
                 │       UNIT         │
                 └─────────┬───────────┘
                           │
                           │ control signals
                           ▼
┌──────────┐        ┌─────────────┐        ┌──────────┐
│   PC     │───────►│             │◄──────►│   RAM    │
└──────────┘        │   BUS 8-bit │        │ 16 × 8   │
                    │             │        └──────────┘
┌──────────┐        │             │
│ Registers│◄──────►│             │
└──────────┘        └──────┬──────┘
                           │
                           ▼
                    ┌─────────────┐
                    │     ALU     │
                    │ ADD/SUB/    │
                    │ AND/OR      │
                    └─────────────┘
```

Este diagrama representa apenas a estrutura conceitual. As conexões definitivas entre registradores, barramento, memória, ALU e unidade de controle serão especificadas durante a implementação no Logisim.

---

# 5. Componentes arquiteturais previstos

A arquitetura atualmente pressupõe a existência dos seguintes blocos funcionais:

| Bloco                         | Função                               | Status     |
| ----------------------------- | ------------------------------------ | ---------- |
| ALU                           | Operações aritméticas e lógicas      | Confirmado |
| Registrador A                 | Armazenamento de operando/resultados | Provisório |
| Registrador B                 | Armazenamento de segundo operando    | Provisório |
| Instruction Register (IR)     | Armazenamento da instrução atual     | Provisório |
| Program Counter (PC)          | Endereço da próxima instrução        | Provisório |
| Memory Address Register (MAR) | Armazenamento do endereço de memória | Provisório |
| Barramento principal          | Transporte de dados de 8 bits        | Provisório |
| RAM                           | Armazenamento de programa e dados    | Confirmado |
| Unidade de controle           | Geração dos sinais de controle       | Confirmado |
| Sequenciador                  | Geração de T0–T5                     | Confirmado |
| Registradores de flags        | Armazenamento de Z e C               | Confirmado |
| Registrador de saída          | Apresentação de resultados           | Em aberto  |

A existência de alguns registradores é uma consequência natural da arquitetura em desenvolvimento, mas sua necessidade exata, tamanho, forma de conexão e capacidade de carga/leitura deverão ser validados no datapath definitivo.

---

# 6. Barramento

## 6.1 Barramento de dados

**Provisório:** a arquitetura utilizará um **barramento central de 8 bits** para interligar os principais elementos do processador.

O barramento permitirá que diferentes módulos:

* coloquem dados no barramento;
* recebam dados do barramento.

A transferência deverá ser controlada por sinais de habilitação.

A regra fundamental do barramento será:

> Em qualquer instante, no máximo um dispositivo deve dirigir o barramento.

Múltiplos drivers ativos simultaneamente constituem uma condição inválida do sistema.

---

## 6.2 Implementação elétrica do barramento

**Provisório:** a implementação física deverá utilizar buffers/tristate para controlar quais módulos podem conduzir o barramento.

A escolha definitiva dos circuitos integrados e a quantidade necessária de buffers dependerão do datapath final.

---

# 7. Program Counter

**Provisório:** o computador deverá possuir um **Program Counter (PC)** com largura suficiente para representar os 16 endereços do espaço de memória.

Como o espaço de endereçamento é de 4 bits, o PC é esperado como um registrador de:

$$
4\text{ bits}
$$

Funções previstas:

* reset;
* incremento;
* carregamento paralelo, caso necessário à ISA;
* disponibilização do endereço atual para acesso à memória.

O conjunto final de operações do PC ainda deverá ser formalizado.

---

# 8. Instruction Register

**Provisório:** a instrução buscada da memória será armazenada em um **Instruction Register (IR)** de 8 bits.

Como o formato da instrução é:

```text
[ opcode 4 bits ][ operand 4 bits ]
```

o IR deverá disponibilizar esses dois campos separadamente para a unidade de controle e para o datapath.

---

# 9. Memory Address Register

**Provisório:** poderá ser utilizado um **Memory Address Register (MAR)** para armazenar o endereço da memória durante operações de leitura e escrita.

Como o espaço de endereçamento é de 4 bits, o MAR deverá possuir 4 bits.

A necessidade exata do MAR deverá ser confirmada quando o datapath e as micro-operações forem definidos.

---

# 10. Registradores de dados

**Provisório:** a arquitetura deverá possuir registradores internos para armazenar operandos e resultados intermediários.

A configuração atualmente considerada inclui pelo menos:

* registrador A;
* registrador B.

A necessidade de registradores adicionais dependerá da sequência de micro-operações e da implementação final da ISA.

---

# 11. Sequenciamento temporal

**Confirmado:** os seis estados temporais `T0–T5` serão gerados por um sequenciador dedicado.

A implementação física prevista utiliza um **contador Johnson/decade counter 74HC4017**.

O sequenciador deverá garantir uma sequência determinística de estados:

```text
T0 → T1 → T2 → T3 → T4 → T5 → T0
```

A forma exata como o reset e a sincronização dessa sequência serão implementados deverá ser validada durante a construção do circuito.

---

# 12. Clock

**Provisório:** o clock físico deverá ser gerado por circuitos baseados em **NE555**.

O sistema deverá possuir um clock capaz de operar o processador de maneira síncrona.

Também está prevista a possibilidade de utilizar controle manual do clock para depuração, permitindo observar o computador passo a passo.

A frequência definitiva de operação ainda não foi definida.

---

# 13. Unidade de controle: princípio de funcionamento

A unidade de controle deverá receber, no mínimo:

```text
Opcode
Estado temporal
Flags relevantes
```

e produzir os sinais necessários para controlar os módulos do computador.

Conceitualmente:

```text
                ┌──────────────┐
Opcode ────────►│              │
                │   CONTROL    │────► Control signals
State ─────────►│    LOGIC     │
Flags ─────────►│              │
                └──────────────┘
```

A arquitetura prevê o uso de:

* decodificadores;
* portas AND;
* portas OR;
* portas NOT;
* demais elementos de lógica necessários.

A expressão booleana final de cada sinal de controle deverá ser derivada das micro-operações da CPU.

---

# 14. Instruction Set Architecture (ISA)

**Confirmado:** o opcode possui 4 bits e, portanto, existem no máximo **16 opcodes** distintos.

O formato geral das instruções é:

```text
[ OPCODE ][ OPERANDO ]
    4 bits    4 bits
```

**Provisório:** o conjunto possível das 16 instruções está apresentado abaixo:

#### ISA v0.1 proposta

| Opcode | Hex | Mnemônico | Operação | Flags afetadas |
| :--- | :--- | :--- | :--- | :--- |
| 0000 | 0x0 | NOP | Nenhuma operação | — |
| 0001 | 0x1 | LDA addr | A ← RAM[addr] | — |
| 0010 | 0x2 | ADD addr | A ← A + RAM[addr] | Z, C |
| 0011 | 0x3 | SUB addr | A ← A - RAM[addr] | Z, C |
| 0100 | 0x4 | STA addr | RAM[addr] ← A | — |
| 0101 | 0x5 | LDI val | A ← val | — |
| 0110 | 0x6 | JMP addr | PC ← addr | — |
| 0111 | 0x7 | JC addr | Se C=1, PC ← addr | — |
| 1000 | 0x8 | JZ addr | Se Z=1, PC ← addr | — |
| 1001 | 0x9 | AND addr | A ← A AND RAM[addr] | Z |
| 1010 | 0xA | OR addr | A ← A OR RAM[addr] | Z |
| 1011 | 0xB | INC | A ← A + 1 | Z, C |
| 1100 | 0xC | DEC | A ← A - 1 | Z, C |
| 1101 | 0xD | CMP addr | Compara A com RAM[addr] | Z, C |
| 1110 | 0xE | OUT | OUT ← A | — |
| 1111 | 0xF | HLT | CPU entra em estado HALT | — |

>[!IMPORTANT] Tarefa
> Próximo passo: antes de marcar isso como ISA confirmada, devemos pegar cada uma das 16 instruções e 
> escrever suas micro-operações T0–T5. Isso vai nos dizer se a ISA é realmente implementável com o 
> datapath que estamos projetando. Se alguma instrução exigir hardware desnecessário ou criar uma 
> sequência problemática, corrigimos agora, antes de construir o Logisim.

A ISA deve sempre ser definida levando em consideração:

1. capacidade real do datapath;
2. registradores disponíveis;
3. operações da ALU;
4. acesso à memória;
5. controle de fluxo;
6. manipulação das flags;
7. simplicidade da unidade de controle.

A ISA não deve ser definida de maneira independente do datapath, pois cada instrução deverá possuir uma sequência implementável de micro-operações.

---

# 15. Micro-operações

**Em aberto:** a sequência detalhada de micro-operações ainda não foi definida.

Para cada instrução deverão ser especificadas as ações realizadas em cada estado `T0–T5`.

Por exemplo, uma instrução poderá possuir uma estrutura conceitual como:

```text
T0: PC → MAR
T1: RAM[MAR] → IR
T2: PC ← PC + 1
T3: ...
T4: ...
T5: ...
```

O exemplo acima é **ilustrativo**, não uma definição da implementação final.

Depois que as micro-operações forem definidas, será possível derivar os sinais de controle e, posteriormente, a lógica booleana da unidade de controle.

---

# 16. Relação entre arquitetura, ISA e controle

O projeto seguirá a seguinte cadeia de derivação:

```text
Arquitetura
     │
     ▼
Datapath
     │
     ▼
ISA
     │
     ▼
Micro-operações
     │
     ▼
Sinais de controle
     │
     ▼
Lógica da unidade de controle
```

Esses níveis não devem ser confundidos.

A ISA descreve **o que uma instrução significa**.

As micro-operações descrevem **como o hardware realiza a instrução**.

A unidade de controle implementa **os sinais necessários para executar essas micro-operações**.

---

# 17. Implementação em Logisim

**Confirmado:** a primeira implementação do computador será realizada no **Logisim**.

A simulação deverá ser utilizada para:

* desenvolver os módulos;
* testar individualmente cada módulo;
* integrar o datapath;
* validar a ISA;
* validar o sequenciamento;
* validar a unidade de controle;
* executar programas de teste;
* encontrar erros antes da construção física.

A implementação em hardware físico somente deverá ser iniciada após a arquitetura simulada apresentar comportamento suficientemente validado.

---

# 18. Implementação física

**Confirmado:** após a validação no Logisim, o projeto será convertido para hardware físico utilizando componentes lógicos discretos, principalmente da família **74HC**.

A implementação deverá ser realizada em protoboards e dividida em módulos funcionais.

O objetivo é construir o computador progressivamente, testando cada módulo antes da integração completa.

---

# 19. Família lógica

**Provisório:** a implementação física utilizará predominantemente circuitos integrados da família:

```text
74HC
```

A escolha dos componentes específicos ainda está sujeita à validação do projeto elétrico.

Nenhuma lista preliminar de componentes deve ser considerada como Bill of Materials definitiva antes da conclusão da arquitetura e da simulação.

---

# 20. Programação manual

**Confirmado:** a memória deverá poder ser programada manualmente através de hardware externo ao processador.

A interface prevista deverá fornecer:

```text
Endereço: 4 bits
Dados:    8 bits
```

O modo de programação não faz parte das instruções executadas pela CPU; ele constitui uma função de operação do sistema físico.

---

# 21. Interface de saída

**Em aberto:** a forma definitiva de apresentação dos resultados ainda não foi estabelecida.

Uma possibilidade considerada é utilizar LEDs e circuitos de decodificação para exibir valores binários ou decimais, porém isso ainda não constitui uma decisão arquitetural definitiva.

---

# 22. Reset

**Em aberto:** o comportamento completo do reset ainda precisa ser especificado.

A definição deverá estabelecer, no mínimo:

* estado inicial do PC;
* estado inicial dos registradores;
* estado inicial das flags;
* estado inicial do sequenciador;
* estado da unidade de controle;
* comportamento dos sinais de controle durante o reset.

---

# 23. Tratamento das flags

A existência das flags `Z` e `C` está confirmada.

**Em aberto:** ainda é necessário definir precisamente:

* quais instruções atualizam `Z`;
* quais instruções atualizam `C`;
* se operações lógicas alteram flags;
* como `C` será interpretada durante `SUB`;
* se instruções de transferência preservam as flags;
* como as flags serão utilizadas em instruções de controle de fluxo.

---

# 24. Subtração

**Confirmado:** a ALU deverá suportar subtração.

**Provisório:** a implementação será baseada em complemento de dois:

$$
A-B=A+\overline{B}+1
$$

Isso implica que a implementação da ALU deverá incluir uma forma de:

1. inverter os bits de `B`;
2. selecionar entre `B` e `~B`;
3. fornecer o carry-in apropriado ao somador.

A lógica final deverá ser validada no Logisim antes da implementação física.

---

# 25. Representação numérica

**Em aberto:** a especificação ainda não estabelece uma interpretação única para todos os valores de 8 bits.

Dependendo da instrução, um valor de 8 bits poderá ser tratado simplesmente como uma sequência binária ou como um número inteiro.

A interpretação específica de valores negativos e o uso de complemento de dois deverão ser documentados na especificação da ISA quando necessário.

---

# 26. Limitações arquiteturais conhecidas

A arquitetura possui algumas limitações decorrentes de suas próprias escolhas:

### Espaço de memória

Com apenas 4 bits de endereço:

$$
16\text{ posições}
$$

estão disponíveis.

### Tamanho da instrução

Todas as instruções possuem apenas 8 bits.

Isso limita cada instrução a:

```text
4 bits de opcode
4 bits de operando
```

### Número de instruções

Existem apenas:

$$
16
$$

opcodes possíveis.

Essas limitações são intencionais nesta etapa do projeto e fazem parte do caráter educacional e experimental do computador.

---

# 27. Decisões provisórias de implementação

As decisões abaixo são direções de projeto atualmente consideradas, mas **não devem ser tratadas como requisitos imutáveis**:

* utilização de um barramento central de 8 bits;
* utilização de registradores A e B;
* utilização de Instruction Register;
* utilização de Program Counter de 4 bits;
* utilização de Memory Address Register de 4 bits;
* utilização de lógica tri-state para controlar o barramento;
* utilização de 74HC4017 para o sequenciador;
* utilização de NE555 para geração do clock;
* utilização predominante da família 74HC;
* implementação da subtração através de complemento de dois;
* programação da RAM através de multiplexação entre modo `PROG` e modo `RUN`.

Essas escolhas só deverão ser promovidas à categoria **confirmado** quando tiverem sido validadas pelo projeto e pelos testes correspondentes.

---

# 28. Questões em aberto

As seguintes questões ainda precisam ser decididas ou formalizadas:

1. Qual será o conjunto definitivo das 16 instruções?
2. Quais registradores existirão exatamente no datapath?
3. Qual será o fluxo exato de dados entre registradores, ALU, barramento e memória?
4. Quais operações cada registrador poderá realizar?
5. O PC poderá ser carregado paralelamente?
6. Qual será exatamente o papel do MAR?
7. Como será realizado o fetch da instrução?
8. Quais micro-operações ocorrerão em `T0–T5`?
9. Quais sinais de controle existirão?
10. Quais instruções alteram `Z`?
11. Quais instruções alteram `C`?
12. Como `SUB` determinará e armazenará a flag `C`?
13. Como serão implementadas instruções condicionais, caso existam?
14. Haverá instruções de salto absoluto?
15. Haverá instruções de entrada/saída?
16. Qual será a arquitetura definitiva de saída?
17. Qual será o comportamento do reset?
18. Qual será a frequência nominal do clock?
19. Como será implementado o clock manual/passo a passo?
20. Quantos circuitos integrados de cada tipo serão realmente necessários?
21. Qual será a implementação elétrica definitiva do barramento?
22. Como serão evitados estados inválidos e contenção de barramento?
23. Como serão tratados sinais em alta impedância durante a operação?
24. Qual será a organização física das protoboards?
25. Qual será a lista definitiva de componentes (BOM)?

---

# 29. Critério de maturidade da arquitetura

A arquitetura será considerada suficientemente especificada para a implementação completa quando, no mínimo, os seguintes elementos estiverem definidos:

* largura e organização dos dados;
* espaço de endereçamento;
* datapath;
* conjunto de registradores;
* ALU;
* flags;
* formato das instruções;
* ISA completa;
* ciclo de máquina;
* micro-operações de todas as instruções;
* sinais de controle;
* lógica da unidade de controle;
* comportamento de reset;
* comportamento da memória;
* interface de programação;
* testes funcionais do processador no Logisim.

Antes desse ponto, componentes físicos e quantidades de CIs devem ser considerados sujeitos a alteração.

---

# 30. Princípio de desenvolvimento

O desenvolvimento seguirá a ordem geral:

```text
Especificação
     ↓
Arquitetura
     ↓
Datapath
     ↓
ISA
     ↓
Micro-operações
     ↓
Unidade de controle
     ↓
Simulação no Logisim
     ↓
Validação
     ↓
Esquemático elétrico
     ↓
Planejamento das protoboards
     ↓
Construção física
     ↓
Testes e depuração
```

A regra fundamental do projeto é:

> **Não implementar em hardware uma parte da arquitetura que ainda não tenha sido suficientemente especificada e validada em simulação.**

---

# 31. Estado atual do projeto

No momento, o projeto possui as seguintes decisões consolidadas:

```text
Dados                 8 bits
Endereço              4 bits
Memória               16 × 8 bits
Instrução             8 bits
Opcode                4 bits
Operando              4 bits
ALU                   ADD / SUB / AND / OR
Flags                 Zero / Carry
Ciclo                 T0–T5
Controle              Hardwired
Simulação inicial     Logisim
Implementação física  Lógica discreta, predominantemente 74HC
Programação da RAM    Manual
```

Ainda permanecem em desenvolvimento:

```text
ISA completa (testar as micro operações de cada intrução do ISA v0.1)
Datapath definitivo
Conjunto de registradores
Micro-operações
Sinais de controle
Lógica detalhada da unidade de controle
Reset
Interface de saída
Clock definitivo
BOM definitiva
Projeto elétrico
Layout físico das protoboards
```

Este documento deverá ser atualizado à medida que essas questões forem resolvidas.

---

# 32. Histórico de revisão

| Versão | Data       | Alteração                                 |
| ------ | ---------- | ----------------------------------------- |
| 0.1    | 2026-09-17 | Registro inicial da arquitetura conhecida |
| 0.2    | 2026-09-17 | ISA v0.1                                  |
| 0.2.1  | 2026-09-17 | Adicionando tarefas sobre ISA             |

```

Esse documento já estabelece uma distinção importante: **“8 bits, 4 bits de endereço, 16×8, instrução de 8 bits, ALU, flags, T0–T5 e controle hardwired” são arquitetura; 74HC4017, NE555 e determinados registradores são implementação ainda sujeita à validação.** Isso evita que uma escolha feita cedo demais vire uma restrição artificial para o restante do projeto.
```
