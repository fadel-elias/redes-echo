# Plano do trabalho de Redes de Computadores

## Objetivo e requisitos

Implementar um cliente e um servidor TCP em C++17 para a atividade do laboratorio 02 da UFJF. O servidor deve atender clientes simultaneamente usando threads e interpretar os comandos `echo` e `quit`.

O projeto sera compilavel em Windows, Linux e macOS usando CMake e um compilador C++17. Cada sistema gera seus proprios executaveis. A execucao ocorre no terminal, sem interface grafica ou bibliotecas externas de rede.

## Organizacao

- `src/servidor.cpp`: aceita conexoes e inicia uma `std::thread` por cliente.
- `src/cliente.cpp`: conecta, le comandos do terminal e mostra respostas.
- `src/rede.hpp`: concentra inicializacao, fechamento e diferencas entre Winsock e sockets POSIX, alem das operacoes de envio e leitura por linha.
- `CMakeLists.txt`: define a compilacao e as bibliotecas do sistema.
- `testes/`: testes automatizados do protocolo e da concorrencia.
- `.github/workflows/`: compilacao e testes nos tres sistemas.
- `README.md`: identificacao, requisitos, protocolo, execucao e resultados verificados.

## Protocolo proposto

Usar TCP, texto UTF-8 sem BOM e uma mensagem por linha. O emissor termina cada mensagem com LF; o receptor tambem aceita CRLF. Comandos sao escritos em minusculas.

| Requisicao | Resposta | Conexao |
| --- | --- | --- |
| `echo <mensagem>` | A mensagem, preservando seus espacos e acentos | Permanece aberta |
| `quit` | `bye` | Servidor fecha apos enviar a resposta |
| `echo` ou `echo ` | `erro: mensagem ausente` | Permanece aberta |
| Linha vazia ou outro comando, inclusive `quit` com argumentos | `erro: comando invalido` | Permanece aberta |

O primeiro espaco depois de `echo` separa o comando do parametro. Todos os bytes restantes sao a mensagem; uma mensagem composta por espacos e permitida. Quebras de linha nao fazem parte do parametro.

O cliente envia um comando e aguarda uma resposta antes do proximo. O servidor aceita varios comandos na mesma conexao. Nao ha saudacao inicial nem prefixo de nome de usuario. O estado da conexao permanece aguardando comandos apos `echo` ou erro e termina apos `quit` ou desconexao.

O envio deve tratar escritas parciais. A leitura deve acumular bytes ate a quebra de linha e preservar bytes de mensagens seguintes. Uma linha incompleta no momento da desconexao e descartada. Uma falha em um cliente encerra somente seu atendimento e libera seu socket; o servidor continua aceitando conexoes. Em sistemas POSIX, a escrita para uma conexao encerrada nao deve terminar o processo por SIGPIPE.

## Execucao

O servidor escuta em IPv4, por padrao em `127.0.0.1:4444`, com endereco de escuta e porta opcionais. O cliente usa o mesmo destino por padrao, com endereco IPv4 e porta opcionais. O teste inicial usa tres terminais na mesma maquina: um servidor e dois clientes. A execucao entre maquinas pode usar um endereco de rede acessivel e permissao de firewall adequada.

Parametros invalidos, porta ocupada e destino indisponivel devem gerar mensagens claras. O fim da entrada padrao encerra o cliente e fecha seu socket. O servidor permanece em execucao ate ser interrompido pelo operador.

## Verificacao

1. Compilar cliente e servidor nos tres sistemas.
2. Verificar echo simples, acentos, espacos e varias requisicoes na mesma conexao.
3. Verificar comandos invalidos e parametro ausente.
4. Manter um cliente conectado e ocioso enquanto outro recebe respostas.
5. Encerrar um cliente por `quit` e continuar usando o outro; aceitar um novo cliente.
6. Desconectar um cliente inesperadamente e confirmar que o servidor continua funcionando.
7. Enviar uma linha dividida em varias escritas e varias linhas numa unica escrita.
8. Verificar falha de conexao, argumentos invalidos e porta ocupada.

Os testes de integracao usarao apenas bibliotecas padrao do Python como ferramenta de teste; os programas entregues e seu multithreading serao implementados em C++. A matriz do GitHub Actions registrara quais sistemas foram efetivamente verificados.

## Etapas e entrega

1. Preparar CMake e validar a compilacao de programas minimos.
2. Implementar as operacoes de rede portaveis.
3. Implementar uma troca entre cliente e servidor.
4. Implementar o protocolo completo e o atendimento por threads.
5. Executar os testes locais e a matriz de sistemas.
6. Concluir o README e integrar a versao validada na `main`.
7. Entregar pelo Classroom o ZIP dos fontes e da documentacao dessa versao e o link do repositorio publico.

Desenvolver na branch `desenvolvimento`. O ZIP deve excluir `.git`, executaveis e diretorios de compilacao. Formatos adicionais de entrega dependem de orientacao expressa na atividade do Classroom.
