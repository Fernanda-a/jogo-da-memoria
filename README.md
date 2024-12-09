\documentclass{rapport}
\usepackage{lipsum}
\usepackage{gensymb}
\usepackage{listings}
\usepackage{hyperref}
\usepackage{xcolor}
\usepackage{hyperref}
\usepackage{float}
\usepackage{graphicx} % Required for inserting images
\pagestyle{plain}
\title{file title} %title of the file

\begin{document}

%----------- Report information ---------
\logo{logos/ufrj-logo-13}
\uni{\textbf{Universidade Federal do Rio de Janeiro}}
\ttitle{Sistemas Digitais - Relatório 2\\Jogo da Memória para FPGA} %title of the file
\subject{Adaptação do jogo da memória para um FPGA} % Subject name


\professor{\textsc{Pedro Cruz}} % information related to the professor

\students{\textsc{Fernanda O. Paschoal da Silva}\\
          \textsc{Igor Mota da Costa}\\
          \textsc{Nicolas Viana do E. Santo}} % information related to the students

%----------- Init -------------------
        
\buildmargins % display margins
\buildcover % create the front cover of the document
\toc % creates the table of contents

%------------ Report body ----------------
\section{Introdução}
O presente trabalho tem por objetivo desenvolver um ``Jogo da Memória'' em VHDL (VHSIC Hardware Description Language, onde VHSIC significa Very High Speed Integrated Circuit) para ser executado em uma placa FPGA (field-programmable gate array), um dispositivo lógico programável que suporta a implementação de circuitos digitais.
Para tal, foram utilizados 4 módulos, disponibilizados pela disciplina, que, quando integrados, permitem o funcionamento do conjunto: visor LCD e teclado periférico conectado através da entrada PS/2, cujos eram requisito mínimo desse trabalho.
\\\\
Segue abaixo os módulos:

% Este trabalho tem como objetivo o desenvolvimento de um "Jogo da Forca".
% O projeto foi feito utilizando o Kit Xilinx Spartan3 e um teclado conecatdo à placa FPGA através de um cabo ps2. Para isso, foram utilizados 4 módulos previamente desenvolvidos para o funcionamento do teclado e do visor LCD:
 \begin{itemize}
   \item ps2\_rx.vhd
   \item fifo.vhd
   \item kb\_code.vhd
   \item key2ascii.vhd
   \item lcd.vhd
 \end{itemize}
 
 As conexões dos 4 módulos seguem a seguinte hierarquia:
 \begin{figure}[H]
\centering
\includegraphics[scale=1]{imgs/arquiovos.png}
\caption{Hierarquia - ISE Project Navigator}
\label{fig:diagrama}
\end{figure}
\newpage
E suas interconexões podem ser expressas pelo seguinte diagrama:
  \begin{figure}[H]
\centering
\includegraphics[scale=0.4]{imgs/modulos_ligados.png}
\caption{Diagrama de arquivos}
\label{fig:diagrama}
\end{figure}
    
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\section{Desenvolvimento}

%%%%%%%%%%%%%%%%% PS2.VHD
\subsection{PS/2}
O módulo ps2-rx realiza o contato direto com o teclado fazendo retornos de 8 bits para as teclas pressionadas. O conector ps2 é um conector mini-DIN de 6 pinos usado para conectar teclados e mouses a um sistema computacional (Figura \ref{fig:ps2-conector}). 

\begin{figure}[H]
\centering
\includegraphics[scale=0.8]{imgs/ps2.jpg}
\caption{conector ps2}
\label{fig:ps2-conector}
\end{figure}

Para a comunicação com o teclado, as entradas ps2d e ps2c são passadas para o programa representando os pinos 1 e 5 respectivamente. 



%%%%%%%%%%%%%%%%% FIFO.vhd
\subsection{FIFO}
O módulo \textit{FIFO} representa uma estrutura de dados que guarda valores em fila, ou seja, o primeiro dado inserido é o primeiro a ser retirado  (first in first out), e é utilizada para armazenar os vetores de 8 bits que referenciam a tecla lida. 
(Figura \ref{fig:fifo}). 

\begin{figure}[H]
\centering
\includegraphics[scale=0.7]{imgs/Fifo_queue.png}
\caption{representação de uma fifo}
\label{fig:fifo}
\end{figure}

A \textit{FIFO} utilizada possui 4 bits de espaço, o que gera um atraso de 4 teclas à cada leitura. Porém, esse problema foi contornado pela lógica do jogo e o código não precisou ser alterado.




%%%%%%%%%%%%%%%%% Kb_code.vhd
\subsection{kbcode}
O módulo kbcode foi utilizado para a leitura das teclas pressionadas no teclado e armazenamento desses dados na fifo. Para isso, ele faz uso dos módulos fifo e ps2\_rx previamente analisados. 


%%%%%%%%%%%%%%%%% key2ascii.vhd
\subsection{key2ascii}
O módulo key2ascii foi utilizado para a conversão das teclas salvas em fifo para o código ASCII. Basicamente, ele recebe o vetor de 8 bits que representa a leitura direta de uma tecla pressionada e o converte para o código ASCII correspondente, devolvendo o vetor resultado. Este módulo é necessário para a utilização do visor LCD.


%%%%%%%%%%%%%%%%% lcd.vhd
\subsection{LCD}

Este foi o único código modificado. O módulo lcd é responsável pela comunicação com o display LCD da placa FPGA e, além disso, toda a lógica do jogo, juntamente com a integração dos módulos e chamadas das funções, ocorreu nesse arquivo.
A comunicação com o display LCD ocorre através de uma máquina de estados que controla um ponteiro que percorre sequencialmente as posições da tela exibindo no visor o seu valor salvo.
\begin{figure}[H]
\centering
\includegraphics[scale=0.7]{imgs/inicio.png}
\caption{tela inicial Memory Game}
\label{fig:telainicial}
\end{figure}


\section{Jogo da Memória}
\subsection{Variáveis}
Para o funcionamento do jogo, foram utilizados os seguintes sinais e variáveis:\newline
 \begin{itemize}
   \item start : indica o início da ``partida''
   \item vidas : indica o número de vidas restante
   \item counter : contador para a troca de letras a cada 3,5 segundos
   \item word\_seq: array que guarda a sequência de letras.
   \item state : variavel para alternar o estado da sequência de letras que aparecerá no display
   \item word : vetor de 4 bits
            
 \end{itemize}
\subsection{Lógica}
Para implementação de um jogo da memória utilizando apenas o display LCD, teclado e a memória interna padrão da placa FPGA, foi preciso pensar em uma lógica que não requisitasse muita memória do sistema e que considerasse também a limitação de espaço do display (suporta 25 caracteres sendo os outros 5 resevardos para configuração do display). Nesse panorama, foi prototipada, para melhor visualização, uma máquina de estados que separesse em etapas, os processos a serem executados:

\begin{figure}[H]
\centering \includegraphics[scale=0.3]{imgs/estadosjogo.png}
\caption{Diagrama Lógica do Jogo}
\label{fig:telainicial}
\end{figure}


Para a sequência de letras, foi implementado um contador, que a cada 3,5 segundos alteraria de letra em letra na sequência. Para fins de teste, foi escolhida uma palavra pequena de apenas 4 letras (PATO), porém, com pequenas alterações, pode-se adicionar sequências maiores.

Após a sequência, com o sinal de start, a tela é limpa, mostrando apenas os 4 espaços para as 4 letras e o número de vidas restantes (inicialmente 5). A máquina alterna entre estado de leitura e análise da letra. No estado de leitura nada acontece até que uma tecla seja pressionada ( kb\_buf\_empty recebe '0'). Quando isso acontece, ela migra para o estado de análise onde a tecla pressionada pelo usuário é salva e analisada.

Se a letra informada for condizente com a esperada da sequência, ela aparece na tela e o verificador ``word'' recebe 1 no índice do espaço da letra, porém, caso não, o número de vidas decai em 1.

Se, após preenchidos todos os espaços e  a sequência estiver correta (word  = ``1111''), a máquina passa para o estado ``ganhou'' e a frase ``VOCE GANHOU'' aparece na tela. Caso não e o número de vidas acabe, passa para o estado ``perdeu'' e aparece a frase ``VOCE PERDEU'' na tela.

\begin{figure}[H]
\centering
\includegraphics[scale=0.7]{imgs/sd.png}
\caption{Tela LCD final jogo ``GANHOU''}
\label{fig:lcd}
\end{figure}


 



% \section{Funcionamento}

% Para o funcionamento da forca, fizemos uma analogia com uma máquina de estados e desenvolmos o código vhdl para executá-la. A máquina é dividida em 5 estados assim como representado na Figura \ref{fig:diagramajogo}.

% %%%%%%%%%%%%%  DIAGRAMA DE ESTADOS 
% \begin{figure}[H]
% \centering
% \includegraphics[scale=0.6]{diagramajogo.jpeg}
% \caption{Diagrama de estados}
% \label{fig:diagramajogo}
% \end{figure}

% No estado inicial uma frase é mostrada na tela (Figura \ref{fig:telainicial}) e permanece estática aguardando o sinal de start dado pelo jogador e para isso basta que ele aperte qualquer tecla. 
% obs.: a tecla pressionada para dar o sinal de start não é salva e não afeta a pontuação do jogo.

% \begin{figure}[H]
% \centering
% \includegraphics[scale=0.1]{inicio.jpg}
% \caption{tela inicial}
% \label{fig:telainicial}
% \end{figure}

% Após o sinal de start, uma nova tela mostra 5 espaços para as 5 letras da palavra além do número de vidas restantes (inicialmente 5). A máquina alterna entre os estados de leitura e análise da letra. No estado de leitura nada acontece até que uma tecla seja pressionada ( kb\_buf\_empty recebe '0'). Quando isso aconte, ela migra para o estado de análise onde a tecla pressionada pelo usuário é salva e analisada. \\
% Se a letra salva estiver na palavra, ela é revelada nos 5 espaços inicialmente vazios, se a letra não estiver na palavra, o número de vidas é diminuído em 1 unidade. Se a palavra não for encontrada (palavra\_certa diferente de "1111") e o usuário ainda tiver vidas restantes (vidas $>$ 0), então a máquina retorna para o estado de leitura e realiza esse ciclo novamente. 
% \\
% Se o usuário completar a palavra antes de perder suas vidas (palavra\_certa = "1111"), a máquina passa para o estado "ganhou" e a frase "Você Ganhou" aparece na tela (Figura \ref{fig:ganhou}) terminando o jogo e deixando a tela estática até um possível sinal de reset.
% \begin{figure}[H]
% \centering
% \includegraphics[scale=0.1]{ganhou.jpg}
% \caption{tela ao ganhar o jogo}
% \label{fig:ganhou}
% \end{figure}

%  Se o número de vidas chegar a zero antes do usuário completar a palavra (vidas = 0), a máquina passa para o estado "perdeu" e a frase "Você Perdeu" aparece na tela (Figura \ref{fig:perdeu}) terminando o jogo e deixando a tela estática até um possível sinal de reset.
% \begin{figure}[H]
% \centering
% \includegraphics[scale=0.1]{perdeu.jpg}
% \caption{tela ao perder o jogo}
% \label{fig:perdeu}
% \end{figure}

% Em  qualquer estado, o sinal de resest ativo (rst = 1) faz o jogo voltar ao estado inicial.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
\section{Conclusão}
    O desenvolvimento do presente trabalho enfrentou diversas dificuldades durante sua construção. A escolha do projeto respondeu às necessidades do trabalho (necessariamente integrar display e teclado) e às limitações dos dispositivos.

    
    Após o desenvolvimento dos códigos e dezenas de testes, o jogo funcionou corretamente e seu formato possibilita aumento do tamanho da palavra escolhida, porém é necessário alterar diversas partes do código.

    
    Como melhoria, seria ideal otimizar o código para esse tipo de mudnça, tornando possivel trabalhar uma lista de sequências, fazendo várias rodadas de jogo. Porém, para isso, é necessário também aumentar a memória disponível. Ademais, o tempo para a troca entre as letras da palavra é maior do que o pretendido devido ao delay para a escrita das palavras no display lcd, uma outra melhoria seria procurar melhorar esse tempo.

    
    Por fim, apesar dos desafios, o trabalho atingiu os objetivos propostos, bem como trabalhou a equipe de forma a se organizar e trabalhar em comunhão vizando a conclusão de um projeto dentro do acordado.
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

\section{Bibliografia}
\begin{enumerate}
    \item WIKIPÉDIA. PS/2. Disponível em: https://pt.wikipedia.org/wiki/PS/2. Acesso em: 26 nov. 2024.
    \item WIKIPÉDIA. FIFO. Disponível em: https://pt.wikipedia.org/wiki/FIFO. Acesso em: 26 nov. 2024.
   \item GILBERTO, Álvaro; UFRJ - Grupo de Teleinformática e Automação. EEL480 - Arquitetura de Computadores. 
   
   Disponível em: https://www.gta.ufrj.br/ensino/EEL480/index.html. 
   Acesso em: 12 nov. 2024.

 \end{enumerate}
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%
 \section{Códigos}
 Os códigos estão disponíveis em \textcolor{blue}{\href{https://github.com/anbubus/jogo-da-memoria}{github.com/anbubus/jogo-da-memoria}}.
 
\end{document} 
