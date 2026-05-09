# Guia de Padronização: Armazenamento de CPF e CNPJ (Edição 2026)
Para desenvolvedores e arquitetos de software, a forma como os documentos brasileiros são armazenados impacta diretamente a performance, a integridade dos dados e a conformidade legal. Com a implementação do CNPJ Alfanumérico em 2026, as práticas antigas (como salvar em campos numéricos) tornaram-se obsoletas e tecnicamente incorretas.
## 1. A Regra de Ouro: Armazenamento "Raw" (Bruto)
A premissa fundamental de banco de dados é a separação entre dado e apresentação. Máscaras de exibição (pontos, traços e barras) pertencem à camada de UI (Front-end), não à persistência.

* Vantagem: Redução de espaço em disco e facilidade em buscas e indexação.
* Ação: Remova caracteres especiais antes do INSERT.

## 2. Definição do Tipo de Dado (Data Type)
Embora o CPF contenha apenas números, o CNPJ está mudando. Por isso, a padronização exige o uso de cadeias de caracteres.

| Documento | Padrão Atual | Tipo Recomendado | Tamanho | Motivo |
|---|---|---|---|---|
| CPF | Numérico | VARCHAR ou CHAR | 11 | Preservação de zeros à esquerda. |
| CNPJ | Alfanumérico | VARCHAR | 14 | Compatibilidade com o padrão 2026 (letras e números). |

Nota Técnica: Evite BIGINT. Além de descartar o zero inicial (ex: 01.234...), campos numéricos causarão erro fatal ao tentar processar os novos CNPJs que contêm letras.

## 3. Implementação Prática## A. Estrutura SQL
Ao criar suas tabelas, utilize o seguinte padrão:

CREATE TABLE organizacoes (
    id UUID PRIMARY KEY,
    razao_social VARCHAR(255) NOT NULL,
    -- VARCHAR(14) para suportar os novos caracteres alfanuméricos
    cnpj VARCHAR(14) NOT NULL UNIQUE, 
    criado_em TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

## B. Limpeza de Dados (Normalização)
No back-end (Node.js, Python, PHP, etc.), utilize Expressões Regulares (Regex) para garantir que apenas o conteúdo essencial chegue ao banco.
Exemplo em JavaScript/Node.js:

const salvarDocumento = (inputUsuario) => {
    // Remove tudo que NÃO for letra ou número (mantendo o novo CNPJ)
    const docNormalizado = inputUsuario.replace(/[^a-zA-Z0-9]/g, "").toUpperCase();
    
    if (docNormalizado.length === 14) {
        // Enviar para o banco como VARCHAR(14)
        return db.insert(docNormalizado);
    }
};

## 4. Validação e Segurança
A conformidade não é apenas técnica, mas legal:

   1. Validação de Dígito: Antes de salvar, aplique o algoritmo de validação (Módulo 11) para evitar dados "lixo".
   2. LGPD: Documentos são dados pessoais sensíveis. Garanta que o acesso a esses campos seja logado e que a tabela utilize criptografia em repouso (Encryption at Rest).

------------------------------
## Fontes e Referências Normativas
Para fins de documentação de projeto ou auditoria, utilize as fontes abaixo:

   1. Normativa Federal (CNPJ Alfanumérico):
   * Instrução Normativa RFB nº 2.229/2024: Estabelece a implementação do CNPJ alfanumérico a partir de julho de 2026 para ampliar a disponibilidade de números.
   2. Normas de Estruturação de Dados:
   * ABNT NBR ISO/IEC 9075: Define os padrões internacionais para linguagens de banco de dados SQL, reforçando a atomicidade e integridade dos dados.
   3. Segurança e Privacidade:
   * Lei nº 13.709/2018 (LGPD): Rege o tratamento de dados pessoais no Brasil, exigindo segurança na custódia de documentos.
      * ISO/IEC 27001: Padrão global para Sistemas de Gestão de Segurança da Informação (SGSI).
   
