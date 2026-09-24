# Estudos-Desafio1-Modulo5-Professor
Este trabalho transforma o banco relacional de uma universidade em um modelo dimensional (esquema em estrela), voltado à análise sob a ótica do professor


1 Introdução
Este trabalho parte de um banco de dados relacional de uma universidade, com as entidades Departamento, Professor, Disciplina, Curso e seus relacionamentos. A partir desse modelo, o desafio consiste em transformar essa estrutura normalizada em um modelo dimensional (esquema em estrela), voltado à análise de dados sob a ótica do professor. Para isso, foi definida uma tabela fato (Fato_Professor) e suas dimensões associadas (Professor, Departamento, Disciplina, Curso e Data), com o objetivo de possibilitar análises como carga horária, quantidade de disciplinas ministradas e atuação de coordenadores, sem considerar dados de alunos. O modelo foi então implementado em script SQL, documentado em diagrama, e populado com dados fictícios para viabilizar sua análise em ferramentas de Business Intelligence, como o Power BI.


2 Execução
Partindo da imagem com o modelo proposto no desafio, foi utilizada uma ferramenta de criação de banco de dados pela web (sqlDBM) para a criação das tabelas dimensões e fato. As tabelas criadas tem as seguintes colunas associadas e desenvolvidas na plataforma sqlDBM:

a) Tabelas dimensão

Dim_Professor: SK_Professor, idProfessor (chave natural). 

Dim_Departamento: SK_Departamento, idDepartamento, Nome, Campus, idProfessor_coordenador.

Dim_Disciplina: SK_Disciplina, idDisciplina, mais nome_disciplina e carga_horária.

Dim_Curso: SK_Curso, idCurso, Departamento_idDepartamento, mais nome_curso e nível.

Dim_Data: SK_Data, data_completa, dia, mês, nome_mês, trimestre, semestre, ano, dia_da_semana.

b) Tabela Fato_Professor

Campo	Tipo	Origem / observação

SK_Professor	FK	Dim_Professor (de idProfessor)

SK_Departamento	FK	Dim_Departamento (de Professor.Departamento_idDepartamento)

SK_Disciplina	FK	Dim_Disciplina (de Disciplina.idDisciplina)

SK_Curso	FK	Dim_Curso (de Disciplina & Curso.Curso_idCurso)

SK_Data_Oferta	FK	Dim_Data (campo assumido, já que o modelo relacional não tem datas)

SK_Coordenador	FK (opcional)	Dim_Professor com outro papel, vindo de Departamento.idProfessor_coordenador

qtd_disciplinas	medida	Sempre 1 por linha, então somar dá o total de disciplinas do professor

qtd_pre_requisitos	medida	Contagem dos pré-requisitos da disciplina, vinda de Pré-requisitos das disciplinas

carga_horaria	medida (assumida)	Horas da disciplina.

flag_coordenador	indicador (0/1)	Se o professor é o coordenador do departamento


3 Pontos de atenção
•	Neste modelo, o foco é o professor, sem análise ou inclusão dos dados de alunos. 

•	Nomes, datas e cargas horárias são inventados e estão presentes nas tabelas no excel com limitação de até 10 linhas. Os dados presentes na planilha fictícia foram importados para o Power BI  apenas para a visualização das tabelas e para a realização da modelagem de dados, sem apresentação de nenhuma análise mais aprofundada no módulo gráfico de visualização do programa da Microsoft.

Tabelas x Qtde de linhas criadas no excel:
fato_professor	10 linhas
Dim_professor	10 linhas
dim_departamento	5 linhas
dim_disciplina	10 linhas
dim_curso	6 linhas
dim_data	8 linhas

•	Consistência: todas as chaves SK_... da tabela fato existem nas dimensões e a chave composta da fato não se repete. Cada professor aparece com o departamento correto, e flag_coordenador vale 1 quando o professor é o coordenador do seu departamento.

•	Formato: cada aba está formatada como Tabela do Excel, com o mesmo nome da aba, o que facilita a importação no Power BI. As colunas têm exatamente os nomes do script SQL.

•	Relações no Power BI: Foi ligada cada dimensão à fato pela chave SK (uma dimensão para muitos registros da fato), como em dim_data[SK_Data] → fato_professor[SK_Data_Oferta]. A ligação entre dim_professor e SK_Coordenador é a segunda relação com a mesma dimensão. O Power BI só permite uma relação ativa entre duas tabelas, então pode ser deixada como inativa com a utilização do  USERELATIONSHIP nas medidas em que quiser o papel de coordenador, conforme explicado no curso.

•	Granularidade - Cada linha representa um professor ministrando uma disciplina, em um curso, em uma data de oferta. Essa granularidade sai da relação Professor → Disciplina e da tabela associativa Disciplina & Curso, que resolve o N:N entre disciplina e curso.


4 Sobre as chaves utilizadas no desafio: Chaves Primária (PK), Estrangeira (FK) e Substituta (SK)

As chaves utilizadas neste desafio descrevem três situações distintas, a PK e FK dizem qual papel a coluna tem dentro da tabela, enquanto a SK diz como a chave foi criada.

PK (Primary Key, chave primária) - É a coluna, ou o conjunto de colunas, que identifica cada linha da tabela de forma única. Ela não pode ser nula nem repetida.
•	Nas dimensões, a PK é a própria chave substituta. Em DIM_PROFESSOR, por exemplo, é SK_Professor.
•	Na fato, a PK é composta pelas cinco chaves SK_Professor, SK_Departamento, SK_Disciplina, SK_Curso e SK_Data_Oferta. Ela garante que a mesma combinação de professor, disciplina, curso e data de oferta não apareça duas vezes. Isso é a granularidade da tabela.

FK (Foreign Key, chave estrangeira)
É uma coluna que aponta para a PK de outra tabela. Ela cria a ligação entre as tabelas e garante a integridade referencial: o banco não deixa gravar na fato um SK_Curso que não exista em DIM_CURSO.
Na fato, todas as chaves SK_... são FK. Por isso as cinco chaves da PK aparecem como PK, FK no diagrama, porque fazem os dois papéis ao mesmo tempo. SK_Coordenador é só FK, porque pode ficar vazio e por isso não entra na PK.

SK (Surrogate Key, chave substituta)
É uma chave artificial, um número inteiro sequencial gerado pelo próprio banco AUTO_INCREMENT), sem significado para o negócio. Ela se opõe à chave natural, que vem do sistema de origem (idProfessor, idCurso). 
Usar SK no lugar da chave natural tem quatro vantagens:
•	Independência da origem: se o sistema de origem mudar ou reaproveitar códigos, o data warehouse continua estável.
•	Histórico: o mesmo professor pode ter várias linhas na dimensão, uma para cada mudança relevante, como troca de departamento ou de titulação. Cada versão recebe uma SK diferente, e a chave natural se repete.
•	Desempenho: junções por um inteiro pequeno são mais rápidas e ocupam menos espaço na fato, que costuma ser a maior tabela do modelo.
•	Casos especiais: dá para criar uma linha "desconhecido" ou "não se aplica" com SK própria, em vez de deixar nulos na fato.

Exemplo em DIM_PROFESSOR:

SK_Professor	idProfessor	titulacao

1	             10	         Mestre

2	             10	         Doutor

O idProfessor 10 é a mesma pessoa. A SK 1 representa o período em que era mestre, e a SK 2 o período depois do doutorado. Uma oferta antiga na fato aponta para a SK 1 e uma recente aponta para a SK 2.

Como funcionam juntas
A chave SK_Professor é PK em DIM_PROFESSOR e FK em FATO_PROFESSOR. Nas consultas e no Power BI, a fato se liga às dimensões por ela:
sql
SELECT d.Nome AS departamento, SUM(f.qtd_disciplinas) AS total_disciplinas
FROM fato_professor f
JOIN dim_departamento d ON d.SK_Departamento = f.SK_Departamento
GROUP BY d.Nome;


5 Conclusão
Este desafio permitiu compreender, na prática, a transição de um modelo relacional normalizado para um modelo dimensional em esquema estrela (star-schema), incluindo a definição de granularidade, o uso de chaves substitutas (SK) e a construção de uma tabela fato com suas dimensões. Mais do que estruturar tabelas, o exercício reforçou a lógica por trás da modelagem orientada à análise, essencial para consultas eficientes e para a integração com ferramentas de BI, como o Power BI. Para a formação em Data Analyst, essa experiência agrega o entendimento de como dados operacionais se transformam em dados analíticos, competência central para quem projeta e interpreta dashboards e relatórios de negócio.
