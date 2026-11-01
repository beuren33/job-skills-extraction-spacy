# Extração de Tecnologias em Vagas com spaCy

Este projeto identifica automaticamente quais tecnologias aparecem na descrição de vagas da área de tecnologia, usando o spaCy com um sistema de regras em vez de um modelo treinado. A ideia é montar um dicionário de termos técnicos (um gazetteer) a partir das próprias vagas e usar o EntityRuler do spaCy para marcar cada termo encontrado no texto com o rótulo TECH. Um exemplo prático seria ler a responsabilidade "Learn and apply .NET Framework and Core fundamentals; Support team in building ASP.NET MVC web applications" e devolver .NET Framework e ASP.NET MVC como tecnologias da vaga.

## Como funciona

Cada vaga do dataset traz uma lista de habilidades (Skills) e de palavras-chave (Keywords) separadas por ponto e vírgula, e o primeiro passo foi transformar cada uma dessas colunas numa lista Python. Juntando todas as listas saíram 20.832 termos, dos quais 2.570 eram únicos. Muitos desses termos eram variações do mesmo item, como ".NET Core basics" e ".NET Core fundamentals", então uma função tirou sufixos como "basics", "fundamentals", "advanced", "expert" e "proficiency", e depois da limpeza ficaram 2.083 termos únicos.

Como o texto corrido costuma abreviar o que a lista formal escreve por extenso (o texto diz "AI models", enquanto a habilidade formal é "Machine Learning"), foram adicionadas à mão 10 variações, como "ML pipelines", "deep learning models" e "version control", chegando a 2.093 padrões. Com eles foi criado um pipeline em branco do spaCy (spacy.blank("en")) com um EntityRuler, onde cada termo vira um padrão com o rótulo TECH. Por ser baseado em regras o método não precisa de treino e a saída é totalmente explicável, já que cada entidade marcada vem de um termo do dicionário, mas só encontra o que está na lista.

## Dados

A base é o Job Descriptions 2025, Tech and Non-Tech Roles (adityarajsrv/job-descriptions-2025-tech-and-non-tech-roles, do Kaggle), arquivo 'job_dataset.csv', com 1.068 vagas. As colunas usadas foram Title, ExperienceLevel, YearsOfExperience, Skills, Responsibilities e Keywords. Os termos do dicionário vêm de Skills e Keywords, e a marcação é aplicada sobre o texto de Responsibilities.

## Resultados

No teste com a primeira vaga (.NET Developer) o pipeline marcou .NET Framework, ASP.NET MVC, SQL queries, Entity Framework, version control e Git, e nas cinco primeiras vagas ele encontrou as tecnologias principais de cada uma, como C#, ASP.NET MVC, Razor e LINQ na segunda. Aplicado nas 1.068 vagas, o pipeline encontrou em média 2 tecnologias por vaga, com no máximo 10.

| Métrica | Valor |
| --- | --- |
| Vagas processadas | 1.068 |
| Padrões no dicionário | 2.093 |
| Média de tecnologias por vaga | 2,0 |
| Máximo em uma vaga | 10 |

Como não há uma anotação manual para comparar, o resultado não foi medido com precision e recall, só inspecionado. O principal limite é que o método só encontra o que está no dicionário e depende de a escrita do texto bater com a da lista.

## Como rodar

Instale as dependências:

```bash
pip install -r requirements.txt
```

Depois abra o notebook e execute as células em ordem:

```bash
jupyter notebook notebook/ner_tecnologias.ipynb
```

O notebook baixa a base do Kaggle pelo kagglehub na primeira execução.

## Estrutura do projeto

```
job-skills-extraction-spacy/
├── notebook/
│   └── ner_tecnologias.ipynb   # gazetteer, EntityRuler e contagem por vaga
├── requirements.txt
└── .gitignore
```

## Observações e próximos passos

Como próximos passos, vale anotar manualmente um conjunto de vagas para medir precision e recall, tratar variações de escrita com o PhraseMatcher em minúsculas, e treinar um modelo de NER do spaCy usando as marcações do EntityRuler como dados iniciais, para encontrar tecnologias que não estão no dicionário.
