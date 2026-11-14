# dio-security-analyst

## Descrição do Projeto

Nesse projeto eu implementei uma skill voltada a analise de vulnerabilidades no DIO Agent, escolhi implementar essa skill devido ao meu interesse na identificação, análise e correção de vulnerabilidades em aplicações e sistemas.  
Para realizar tal feito utilizei os seguintes recursos:

| Componentes | Utilizados |
| --- | --- |
| Editor de Código | Visual Studio Code |
| Harness | Antigravity |

O objetivo é permitir que o agente seja capaz de auxiliar em atividades como:
- Análise de código-fonte para buscar possíveis vulnerabilidades
- Identificação de falhas relacionadas a XSS, SQL injection e controle de acesso.
- Análise de configurações e dependências vulneráveis.
- Classificação de vulnerabilidades encontradas utilizando referências como CWE, CVE e CVSS.
- Avaliação de riscos considerando probabilidade e impacto.
- Sugestão de correções e validação das medidas implementadas.  

## Arquivos Alterados

Para possibilitar que o assistente realiza-se essas atividades, criei e alterei os seguintes arquivos  

### SKILL.md

É onde eu defini, o objetivo por trás do que agente faz e o fluxo das suas ações, fluxo esse que ajuda a deixar mais claro para o estudante o retorno dado pelo agente. Essas etapas do fluxo de trabalho dele consistem em:

- Contextualização.
- Identificação.
- Evidencias.
- Avaliação.
- Correção.
- Validação.
- Relatório.

### security-basics.md

Esse arquivo foi criado com intuito de passar as definições básicas de segurança para o DIO Agent, nele sinalizei e descrevi conceitos fundamentais como vulnerabilidades, ameaças, riscos, princípios de segurança e as boas praticas em que ele devera se basear em todas as suas analises de código.  

### persona.md

Eu editei esse arquivo adicionando a ele a maneira como o agente deve agir, sempre que ele identificar que uma pergunta relaciona a analise de vulnerabilidades for feita, possibilitando assim que a experiência do aluno seja mais proveitosa do que somente receber um aglomerado de textos mostrando o que ele deve fazer.  

### AGENTS.md

Modifiquei esse arquivo, possibilitando portanto que a skill fosse utilizada pelo agente, mostrando pra ele a localização da skill dentro dos arquivos, o nome dela e quando ele deve utilizá-la.

## Teste Pratico

Para que fosse possível compreender as mudanças que o agente teria, após realizar a instalação do Antigravity no terminal, dentro da pasta do DIO Agent, utilizando o seguinte comando:
```
curl -fsSL https://antigravity.google/cli/install.sh | bash
```
Assim que a instalação terminar, e possível utilizá-lo digitando o seguinte comando no terminal:
```
agy
```
Desse modo utilizei o seguinte código de python que possui uma falha que possibilita um ataque chamado SQL injection, que se baseia em inserir comando SQL em campos de entrada de dados, como em formulários de login que foi a situação esquematizada, o código é o seguinte:  
```Python
import sqlite3

def fazer_login(usuario, senha):
        conexao = sqlite3.connect('banco_de_dados.db')
        cursor = conexao.cursor()

  
        query = "SELECT * FROM usuario WHERE username = '{usuario}' AND password = '{senha}'"
        
        
        cursor.execute(query)
        resultado = cursor.fetchone()
        conexao.close()

        if resultado:
            return "Acesso Permitido!"
        else: 
            return "Acesso Negado."
```

### Primeiro Teste

Nesse primeiro momento eu pedi para o agente sem inclusão de skill alguma analisar o código, com o seguinte prompt:
```
Atue como um especialista em AppSec. Leia o arquivo app.py nesta pasta. Identifique se existe alguma vulnerabilidade critica, explique como um atacante poderia explorá-la, cite a técnica correspondente no MITRE ATT&CK e reescreva o código corrigindo a falha de forma segura
```
E essa foi a resposta do agente: 
```markdown
Análise de segurança da aplicação no arquivo       
  dio-agent/agent/app.py:                            
  ──────
  ### 1. Identificação da Vulnerabilidade Crítica    
                                                     
  No arquivo dio-agent/agent/app.py, dentro da função
  fazer_login:                                       
  
  • SQL Injection (SQLi) / CWE-89: A consulta SQL foi
  construída por interpolação de strings não         
  sanitizadas diretamente no comando SQL:            
    query = "SELECT * FROM usuario WHERE username =  
    '{usuario}' AND password = '{senha}'"            
  (Mesmo que formatada com f-string ou .format(),    
  concatenar entradas de usuários diretamente na     
  consulta permite que dados fornecidos modifiquem a 
  estrutura sintática da instrução SQL).
  • Armazenamento e Comparação de Senhas em Texto    
  Plano (CWE-256 / CWE-312): A consulta compara a    
  senha fornecida diretamente com o valor persistido 
  no banco, indicando ausência de hashing            
  criptográfico seguro (como Argon2, bcrypt ou       
  PBKDF2).
  ──────
  ### 2. Mecanismo de Exploração (Visão Teórica)     
  
  Quando parâmetros fornecidos pelo usuário são      
  inseridos diretamente no corpo de um comando SQL   
  sem parametrização:
  
  1. Bypass de Autenticação (Tautologia): Um invasor 
  pode fornecer caracteres de controle da sintaxe SQL
  (como ' e operadores lógicos). Ao injetar uma
  condição sempre verdadeira no campo de usuário (por
  exemplo, manipulando a expressão para que resulte 
  em WHERE username = '...' OR '1'='1'), o
  interpretador SQL altera a lógica original da     
  consulta.
  2. Como resultado, a condição avalia como      
  verdadeira para o primeiro registro retornado na   
  tabela usuário, concedendo acesso irrestrito sem o
  fornecimento de credenciais válidas. 
  3. Dependendo da base de dados e do contexto,      
  invasores também podem utilizar técnicas como UNION
  SELECT ou ataques baseados em inferência     
  (Blind/Time-based SQLi) para extrair dados
  confidenciais de outras tabelas.
  ──────
  ### 3. Mapeamento no MITRE ATT&CK
  
  • Tática: Initial Access (Acesso Inicial) / 
  Credential Access (Acesso a Credenciais)
  • Técnica: T1190 - Exploit Public-Facing           
  Application https://attack.mitre.                  
  org/techniques/T1190/                              
  • Adversários exploram vulnerabilidades em     
  aplicações expostas (como injeção de SQL em    
  rotas de autenticação) para burlar controles de
  acesso, executar comandos no banco ou obter    
  credenciais.
  • Técnica Secundária relacionada: T1005 - Data from
  Local System https://attack.mitre.                 
  org/techniques/T1005/ / T1555 - Credentials from   
  Password Stores https://attack.mitre.              
  org/techniques/T1555/.                             
  ──────                                             
  ### 4. Correção Segura do Código
  
  Para mitigar a vulnerabilidade:
  
  4. Consultas Parametrizadas (Prepared Statements): 
  Utilização de placeholders (?) suportados pela     
  biblioteca sqlite3. O driver separa o plano de     
  execução dos dados fornecidos, tratando qualquer   
  entrada estritamente como valor literal.           
  5. Gerenciamento Seguro de Recursos: Utilização de 
  context managers (with) para garantir o fechamento 
  e gerenciamento adequado da conexão.
  6. Verificação de Hash de Senha: Em um ambiente de 
  produção seguro, a tabela armazena apenas hashes   
  salgados gerados por funções de derivação de chaves.
  
  #### Código Corrigido
  
    import sqlite3
    import hmac
    import hashlib
  
    def fazer_login(usuario: str, senha: str) -> str:
        """
        Realiza o login de forma segura utilizando
        consultas parametrizadas
        e tratamento de recursos via context manager.
        """
        # Exemplo utilizando contexto seguro para conexões                                     
        with sqlite3.connect('banco_de_dados.db') as conexao:
            cursor = conexao.cursor()
  
            # Consulta parametrizada com placeholder '?' para prevenir SQL Injection          
            # Busca o hash e o salt associados ao usuário em vez de comparar texto plano   
            query = "SELECT password_hash, salt FROM usuario WHERE username = ?"
            cursor.execute(query, (usuario,))        
            resultado = cursor.fetchone()            
  
            if not resultado:
                return "Acesso Negado."
  
            stored_hash, salt = resultado            
  
            # Derivação de chave usando PBKDF2 com HMAC-SHA256
            computed_hash = hashlib.pbkdf2_hmac(     
                'sha256',
                senha.encode('utf-8'),
                salt.encode('utf-8') if
                isinstance(salt, str) else salt,
                iterations=100_000 
            ).hex()
  
            # Comparação em tempo constante para mitigar Timing Attacks
            if hmac.compare_digest(stored_hash,      
            computed_hash):
                return "Acesso Permitido!" 
            else:
                return "Acesso Negado." 
  
  │ Tip 
  │ Caso a estrutura legada do banco de dados ainda
  │ não utilize hashes, a correção mínima imediata   
  │ para neutralizar o SQL Injection é substituir a
  │ execução pela versão com tupla de parâmetros:
  │
  │   query = "SELECT id FROM usuario WHERE username = ? AND password = ?"
  │   cursor.execute(query, (usuario, senha))
```

### Segundo Teste

Nessa etapa utilizei o agente já com as alterações, e para manter o teste o mais fidedigno possível utilizei o mesmo prompt e esse foi o resultado 
```markdown
Com base nas diretrizes da skill vulnerability-analysis localizada em                                       
  dio-agent/skills/vulnerability-analysis, apresento a análise detalhada de segurança do arquivo              
  dio-agent/agent/app.py.                                                                                     
  ──────                                                                                                      
  ### 1. Contextualização                                                                                     
                                                                                                              
  A função fazer_login tem como objetivo autenticar usuários consultando um banco de dados local SQLite       
  (banco_de_dados.db).                                                                                        
                                                                                                              
  │ Note                                                                               
  │ Para contextualizar melhor o cenário de aplicação:                                                        
  │                                                                                                           
  │ • Este código faz parte de um ambiente didático/laboratório de testes ou será integrado a uma aplicação   
  │ web/API em produção?                                                                                      
  │ • Qual é o nível de privilégio que a aplicação terá no sistema operacional e no banco de dados?           
  ──────                                                                                                      
  ### 2. Identificação de Vulnerabilidades                                                                    
                                                                                                              
  Foram identificadas três vulnerabilidades/más práticas principais no código:                                
                                                                                                              
  1. Risco Crítico — Padrão Inseguro de Consulta SQL / Suscetibilidade a SQL Injection (CWE-89 https://cwe.   
  mitre.org/data/definitions/89.html):                                                                        
  A montagem da consulta SQL utiliza formatação de string direta com placeholders ({usuario}, {senha}). Caso  
  transformada em f-string ou interpolação, entradas de usuários são interpretadas diretamente como comandos  
  SQL pelo interpretador do banco.                                                                            
  2. Risco Alto — Comparação e Armazenamento de Senhas em Texto Claro (CWE-256 https://cwe.mitre.             
  org/data/definitions/256.html / CWE-312 https://cwe.mitre.org/data/definitions/312.html):                   
  A query busca a senha diretamente por valor textual (password = ...), indicando que as senhas não utilizam  
  algoritmo de hashing criptográfico com salt.                                                                
  3. Risco Baixo — Gerenciamento Frágil de Recursos e Falta de Tratamento de Exceções (CWE-404 https://cwe.   
  mitre.org/data/definitions/404.html):                                                                       
  A conexão é fechada manualmente ao final. Se ocorrer uma exceção durante a execução da consulta, o descritor
  de conexão pode não ser liberado adequadamente.                                                             
  ──────                                                                                                      
  ### 3. Evidências no Código                                                                                 
                                                                                                              
  No arquivo dio-agent/agent/app.py:                                                                          
                                                                                                              
  • Linha 8 e Linha 11:                                                                                       
    query = "SELECT * FROM usuario WHERE username = '{usuario}' AND password = '{senha}'"                     
    cursor.execute(query)                                                                                     
  Evidência: A variável query foi estruturada para interpolação direta. Se utilizada como f-string (f"..."),  
  qualquer caractere de controle (como aspas simples ') permite alterar a estrutura lógica da consulta SQL.   
  Além disso, mesmo no formato literal atual, trata-se de um bug de lógica onde os valores dos argumentos não 
  são vinculados.                                                                                             
  • Linha 8 (Armazenamento de Senha):                                                                         
    AND password = '{senha}'                                                                                  
  Evidência: Compara a credencial diretamente com o banco de dados, sem o uso de hash criptográfico seguro    
  (ex.: Argon2, bcrypt).                                                                                      
  • Linhas 4 e 13:                                                                                            
    conexao = sqlite3.connect('banco_de_dados.db')                                                            
    ...                                                                                                       
    conexao.close()                                                                                           
  Evidência: Sem o uso de bloco with ou try...finally, falhas no banco impedem que conexao.close() seja       
  executado.                                                                                                  
  ──────                                                                                                      
  ### 4. Avaliação de Impacto e Severidade                                                                    
                                                                                                              
   Vulnerabilidade                     │ Severidade │ Probabilidade │ Impacto Teórico
  ─────────────────────────────────────┼────────────┼───────────────┼───────────────────────────────────────────
   SQL Injection (CWE-89)              │ Crítica    │ Alta          │ Permite a um atacante burlar o mecanismo
                                       │            │               │ de login (bypass de autenticação) ou
                                       │            │               │ extrair/modificar dados indevidamente.
   Senhas em Plaintext (CWE-256)       │ Alta       │ Média/Alta    │ Em caso de vazamento do banco
                                       │            │               │ banco_de_dados.db, todas as senhas dos
                                       │            │               │ usuários ficam expostas de forma legível.
   Gerenciamento de Recursos (CWE-404) │ Baixa      │ Baixa         │ Possível vazamento de recursos ou
                                       │            │               │ travamento de locks do arquivo SQLite em
                                       │            │               │ picos de erro.
  ──────                                                                                                      
  ### 5. Correção Recomendada                                                                                 
                                                                                                              
  Para mitigar essas falhas:                                                                                  
                                                                                                              
  4. Prepared Statements (Parametrização): Usar placeholders ? suportados pelo driver sqlite3, delegando ao   
  banco a separação estrita entre instrução e dados.                                                          
  5. Hash Criptográfico de Senha: Salvar e validar senhas utilizando algoritmos de derivação de chaves seguros,
  como bcrypt ou hashlib.scrypt.                                                                              
  6. Context Manager (with): Garantir o fechamento automático das conexões.                                   
                                                                                                              
  #### Código Refatorado Seguro:                                                                              
                                                                                                              
    import sqlite3                                                                                            
    import hmac                                                                                               
    import hashlib                                                                                            
                                                                                                              
    def verificar_senha(senha_fornecida: str, hash_armazenado: str, salt: bytes) -> bool:                     
        """Verifica a senha calculando o hash com scrypt e comparando em tempo constante."""                  
        hash_calculado = hashlib.scrypt(                                                                      
            senha_fornecida.encode('utf-8'),                                                                  
            salt=salt,                                                                                        
            n=16384,                                                                                          
            r=8,                                                                                              
            p=1                                                                                               
        ).hex()                                                                                               
        return hmac.compare_digest(hash_calculado, hash_armazenado)                                           
                                                                                                              
    def fazer_login(usuario: str, senha: str) -> str:                                                         
        # Uso do gerenciador de contexto para liberação segura dos recursos                                   
        try:                                                                                                  
            with sqlite3.connect('banco_de_dados.db') as conexao:
                cursor = conexao.cursor()                                                                     
                # 1. Consulta parametrizada com placeholder '?' (Prevenção contra SQL Injection)
                query = "SELECT password_hash, salt FROM usuario WHERE username = ?" 
                cursor.execute(query, (usuario,))                                          
                resultado = cursor.fetchone()                                                                 
                                                                                                              
                if resultado is None: 
                    return "Acesso Negado."                                                                                                                                           
                hash_armazenado, salt_hex = resultado 
                salt = bytes.fromhex(salt_hex)
                                      
                # 2. Validação segura da senha contra o hash armazenado   
                if verificar_senha(senha, hash_armazenado, salt):
                    return "Acesso Permitido!"
                else:
                    return "Acesso Negado."                                                                   
                                                                                                              
        except sqlite3.DatabaseError:   
            return "Erro interno ao processar a autenticação."                                                
  ──────
  ### 6. Validação
                                                                     
  Para validar a eficácia da correção:       
  7. Teste Funcional: Realizar o login com usuário e senha válidos e confirmar o retorno "Acesso Permitido!". 
  8. Teste de Injeção SQL: Tentar autenticar utilizando entradas com caracteres de escape, como ' OR '1'='1 no campo de usuário. Com a parametrização, o SQLite buscará literalmente um usuário com esse texto, retornando "Acesso Negado." sem alterar a sintaxe da consulta.
  9. Teste de Tratamento de Erros: Simular ausência da tabela ou permissão negada no arquivo para garantir que a conexão seja fechada sem travar o processo.

  ──────
  ### 7. Relatório Consolidado
  
  │ Important
  │ Resumo Executivo da Auditoria:
  │
  │ • Arquivo Analisado: dio-agent/agent/app.py
  │ • Status Geral: Vulnerável (Requer refatoração antes de qualquer publicação).
  │ • Ação Imediata: Substituir a concatenação/formatação manual de strings por consultas parametrizadas (?) e adotar hashing criptográfico para armazenamento de senhas.
```
Com essa sendo a alteração que ele propos para o código:  

