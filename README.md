# Compilador.T2

1. Descrição Teórica
Foi desenvolvevido um Analisador Sintático Descendente Recursivo para uma linguagem livre de contexto pré-definida. O analisador foi projetado seguindo a classe de gramáticas LL(1) que se baseia na criação de um procedimento para cada Símbolo Não-Terminal da gramática (P, S, C, V) e o parser toma decisões de fluxo baseando-se apenas no token atual fornecido pelo analisador léxico.
Para garantir que o compilador não parasse ao encontrar o primeiro erro, foi implementada a estratégia de Recuperação de Erros em Modo Pânico. Esta técnica evita que o compilador aborte a execução ao encontrar o primeiro erro, permitindo que ele descarte tokens da entrada até encontrar um "ponto de sincronização" seguro para retomar a análise.

2. Estrutura e Implementação
- Analisador Léxico
Criado no trabalho 1 ele é responsável por transformar a entrada em um token. Implementado como uma Máquina de Estados Finitos através de um switch-case sobre a variável estado.

Tratamento de Arquivo: Foi implementada uma técnica de "Sentinela" na função readFile, alocando bytes extras no buffer para garantir que o Lexer processe corretamente tokens numéricos situados exatamente no fim do arquivo (tratando o problema de ausência de delimitadores como espaço ou quebra de linha).

- Analisador Sintático
Implementa as regras de produção da gramática.
Função match(token): Valida se o token atual corresponde ao esperado e solicita o próximo ao Lexer.
Funções Recursivas: As funções P() (Programa), S() (Comando), C() (Condição) e V() (Valor) implementam a lógica da árvore sintática.

- Gerenciamento de Erros (Panic Mode)
Função panic(): Recebe um conjunto de tokens de sincronização. Em caso de erro, esta função consome e descarta tokens da entrada até encontrar um token pertencente a esse conjunto ou o fim do arquivo (EOF).
Integração: Os blocos else das funções sintáticas foram ajustados para chamar panic() passando o conjunto FOLLOW do não-terminal corrente, garantindo que a análise possa continuar para o próximo comando válido.

3. Funcionamento
A execução inicia-se na função main, que carrega o arquivo programa.txt para a memória. O primeiro token é lido para inicializar a variável token_atual. Em seguida, a função do símbolo inicial P() é chamada.
Durante a descida recursiva, o parser imprime no console as regras gramaticais que estão sendo aplicadas (ex: Regra 3: S -> if...), permitindo a visualização da árvore de derivação.
Se um erro sintático ocorre, a função erro() reporta a linha e o token inesperado, e o controle é passado para o modo pânico. O parser tenta se recuperar e, ao final, o programa informa se a compilação foi um sucesso total ou se terminou com erros.

4. Testes Realizados e Resultados
Foram realizados três cenários de teste para validar a robustez do compilador.

Teste A: Fluxo Feliz (Código Correto)
Entrada: if x >= 10 then y <= 20 Resultado: O compilador reconheceu a estrutura completa. A saída mostrou a aplicação correta das regras (entrada em P, reconhecimento de S, chamada de C para condição, e recursão de S para o corpo do then). Status: Sucesso.

Teste B: Recuperação de Erro Sintático
Entrada: if x > then y <= 20 Cenário: Foi omitido o operando (número ou id) após o operador > na condição. Resultado: O parser detectou o erro na função V() (esperava valor, veio then).
Mensagem: ### ERRO SINTATICO NA LINHA 1: Esperava Identificador ou Numero ###.
Modo Pânico ativado: O parser verificou que THEN estava no conjunto de sincronização de V.
Recuperação imediata: O parser assumiu que a condição terminou e continuou analisando o corpo do then (y <= 20) corretamente. Status: Recuperação bem-sucedida.

Teste C: Validação Léxica
Entrada: x = 10 Cenário: Uso do operador de atribuição = (que foi removido da gramática nesta versão). Resultado: O Analisador Léxico identificou o caractere = isolado (sem ser ==, <=, etc.) e disparou a mensagem: ERRO LEXICO na linha 1: '=' sozinho nao permitido, encerrando a execução de forma controlada. Status: Validação correta.

5. Conclusão
O analisador desenvolvido atende aos requisitos de reconhecer a linguagem proposta e, principalmente, não é interrompido continuando e encontrando novos erros a serem tratados através do Modo Pânico. As dificuldades encontradas com o tratamento de fim de arquivo (EOF) no Lexer foram solucionadas através da implementação de sentinelas no buffer de leitura, garantindo estabilidade na execução.
