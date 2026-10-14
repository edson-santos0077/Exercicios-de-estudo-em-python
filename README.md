Aqui está uma proposta para o ficheiro `README.md` do seu projeto, redigida em Português de Portugal e estruturada com base nas informações e requisitos fornecidos.

---

# Catálogo de Filmes e Séries

## 📌 Visão Geral

Este projeto consiste numa API desenvolvida em FastAPI (ou sistema de linha de comando CLI) para gerir um catálogo pessoal de filmes e séries. O objetivo do sistema é permitir que o utilizador acompanhe o progresso de visualização de séries, compare avaliações entre mídias, gira listas personalizadas e consulte relatórios de consumo. A modelação do sistema é fortemente orientada a objetos (POO), enfatizando conceitos como herança, encapsulamento, validações e composição.

## 🚀 Funcionalidades Principais

* **Cadastro de Mídias**: Permite o registo de filmes e séries com atributos como título, tipo, género, ano, duração e classificação indicativa. O sistema impede a duplicidade de mídias que possuam o mesmo título, tipo e ano.


* **Gestão de Séries e Episódios**: Suporta o registo de temporadas e episódios para a entidade Série. O sistema marca automaticamente a série com o status de "ASSISTIDA" assim que todos os seus episódios estiverem concluídos.


* **Sistema de Avaliações**: Os utilizadores podem avaliar filmes e episódios atribuindo uma nota numérica de 0 a 10. O sistema calcula de forma automática a nota média das séries e a nota geral do catálogo.


* **Histórico e Listas Personalizadas**: Regista a data e hora de conclusão de cada mídia. Permite criar listas personalizadas (ex: "Para assistir", "Favoritos") até um limite definido nas configurações e adicionar/remover mídias das mesmas.


* **Relatórios e Estatísticas**: Geração de relatórios que incluem a média de notas por género, o tempo total assistido por tipo de mídia, o Top 10 das mídias mais bem avaliadas e as séries com o maior número de episódios assistidos. Os relatórios consideram apenas mídias com o status "ASSISTIDO".


* **Configurações Dinâmicas**: Leitura de parâmetros através do ficheiro `settings.json`, que define a nota mínima para uma mídia ser "recomendada", o limite de listas por utilizador e o multiplicador de duração para conversão de tempo.



## 📂 Estrutura de Arquivos e Diretórios

A arquitetura do projeto está organizada em camadas, separando a interface, as regras de negócio e os modelos de domínio:

```text
catalogo_filmes/
│
├── main.py                  # Ponto de entrada da aplicação FastAPI
├── dados.py                 # Módulo exigido para persistência (GerenciadorDados)
├── settings.json            # Ficheiro de configurações exigido no projeto
├── README.md                # O seu ficheiro com a explicação, objetivo e Mermaid
├── requirements.txt         # Lista de dependências (fastapi, pytest)
│
├── api/                     # Camada de Interface (Rotas FastAPI)
│   ├── __init__.py
│   └── rotas.py             # Representa a classe InterfaceFastAPI
│
├── models/                  # Camada de Domínio (Classes principais)
│   ├── __init__.py
│   ├── enums.py             # StatusVisualizacao e TipoMidia
│   ├── midia.py             # Midia, Filme, Serie, Temporada, Episodio
│   └── usuario.py           # Usuario, ListaPersonalizada, RegistroHistorico
│
├── services/                # Camada de Regras de Negócio
│   ├── __init__.py
│   ├── relatorio.py         # RelatorioService (Cálculos de médias e top 10)
│   └── configuracao.py      # Classe Configuracao (Lê o settings.json)
│
└── tests/                   # Diretório para os testes automatizados
    ├── __init__.py
    ├── test_midia.py        # Testes das regras de negócio de Mídias/Episódios
    └── test_relatorios.py   # Testes dos cálculos matemáticos

```

## 📊 Diagrama de Classes (UML)

Abaixo encontra-se a representação da modelação orientada a objetos do projeto, ilustrando os relacionamentos de herança, composição e agregação:


```mermaid
%%{
  init: {
    'theme': 'base',
    'themeVariables': {
      'primaryColor': '#1e293b',
      'primaryTextColor': '#f8fafc',
      'primaryBorderColor': '#334155',
      'lineColor': '#64748b',
      'secondaryColor': '#f1f5f9',
      'tertiaryColor': '#ffffff',
      'fontFamily': 'ui-sans-serif, system-ui, sans-serif'
    }
  }
}%%
classDiagram
    direction TB

    %% ==========================================
    %% CAMADA DE INTERFACE (FASTAPI / ROUTERS)
    %% ==========================================
    class RotasAPI {
        <<Router - FastAPI>>
        +criar_midia(payload: MidiaSchemaCreate)
        +listar_midias()
        +criar_usuario(payload)
        +obter_relatorios()
    }

    class MidiaSchemaCreate {
        <<Pydantic Schema>>
        +titulo: str
        +tipo: str
        +genero: str
        +ano: int
        +duracao_minutos: int
    }

    %% ==========================================
    %% CAMADA DE PERSISTÊNCIA / REPOSITÓRIOS
    %% ==========================================
    class MidiaRepository {
        <<Interface - ABC>>
        +salvar_todos(midias: list)* void
        +carregar_todos()* list
    }

    class MidiaRepositoryJSON {
        <<Repository>>
        -caminho: str
        -_obter_proximo_id(midias: list) int
        +salvar_todos(midias: list) void
        +carregar_todos() list
    }

    class MidiaRepositorySQLite {
        <<Repository>>
        +salvar_todos(midias: list) void
        +carregar_todos() list
    }

    class UsuarioRepository {
        <<Interface - ABC>>
        +salvar(usuario: Usuario)* void
        +carregar(id_usuario: str)* Usuario | None
    }

    class UsuarioRepositoryJSON {
        <<Repository>>
        -caminho: str
        +salvar(usuario: Usuario) void
        +carregar(id_usuario: str) Usuario | None
    }

    MidiaRepository <|-- MidiaRepositoryJSON
    MidiaRepository <|-- MidiaRepositorySQLite
    UsuarioRepository <|-- UsuarioRepositoryJSON

    %% ==========================================
    %% ENUMS E EXCEÇÕES DE DOMÍNIO
    %% ==========================================
    class StatusVisualizacao {
        <<enumeration>>
        NAO_ASSISTIDO
        ASSISTINDO
        ASSISTIDO
    }

    class TipoMidia {
        <<enumeration>>
        FILME
        SERIE
    }

    class DominioError { <<exception>> }
    class ValorInvalidoError { <<exception>> }
    class EstadoInvalidoError { <<exception>> }

    DominioError <|-- ValorInvalidoError
    DominioError <|-- EstadoInvalidoError

    %% ==========================================
    %% MIXINS DE DOMÍNIO
    %% ==========================================
    class AvaliavelMixin {
        <<mixin>>
        -_nota: float | None
        +nota() float | None
        +avaliar(valor: float) void
    }

    class SerializavelMixin {
        <<mixin>>
        +to_dict() dict
        +from_dict(dados: dict)$ Object
    }

    %% ==========================================
    %% CAMADA DE DOMÍNIO (ENTIDADES POO)
    %% ==========================================
    class Midia {
        <<abstract>>
        -_id: int | None
        -_titulo: String
        -_tipo: TipoMidia
        -_genero: String
        -_ano: int
        -classificacao_indicativa: String
        -_elenco: List~String~
        -_status: StatusVisualizacao
        -_data_conclusao: datetime | None
        +id() int | None
        +titulo() String
        +genero() String
        +ano() int
        +tipo() TipoMidia
        +elenco() List~String~
        +data_conclusao() datetime | None
        +status() StatusVisualizacao
        +duracao_minutos()* int
        +alterar_status(novo_status: StatusVisualizacao) void
        +converter_minutos_para_horas(minutos: int, multiplicador: float)$ float
        +__str__() String
        +__repr__() String
        +__eq__(outro: Object) bool
        +__lt__(outro: Object) bool
    }

    class Filme {
        -_duracao_minutos: int
        +duracao_minutos() int
        +__str__() String
    }

    class Serie {
        -temporadas: List~Temporada~
        +duracao_minutos() int
        +nota() float | None
        +atualizar_status() void
        +adicionar_temporada(temporada: Temporada) void
        +__len__() int
        +__str__() String
    }

    class Temporada {
        -_numero: int
        +episodios: List~Episodio~
        +numero() int
        +duracao_minutos() int
        +adicionar_episodio(episodio: Episodio) void
        +calcular_nota_media() float | None
        +__len__() int
        +__repr__() String
        +__str__() String
    }

    class Episodio {
        -_id: int | None
        -_numero_temporada: int
        -_numero_episodio: int
        -_titulo: String
        -_duracao_minutos: int
        -_data_lancamento: date | None
        -_status: StatusVisualizacao
        +id() int | None
        +titulo() String
        +numero_temporada() int
        +numero_episodio() int
        +duracao_minutos() int
        +data_lancamento() date | None
        +status() StatusVisualizacao
        +alterar_status(novo_status: StatusVisualizacao) void
        +__repr__() String
        +__str__() String
    }

    class Usuario {
        -_id_usuario: int | str | None
        -_nome: String
        -_email: String
        -_limite_listas: int
        -_listas: dict~str, ListaPersonalizada~
        -_favoritos: ListaPersonalizada
        -_historico: List~RegistroHistorico~
        +id_usuario() int | str | None
        +nome() String
        +email() String
        +limite_listas() int
        +criar_lista(nome: String) ListaPersonalizada
        +obter_lista(nome: String) ListaPersonalizada
        +adicionar_favorito(midia: Midia) void
        +remover_favorito(midia: Midia) void
        +registrar_historico(midia: Midia, data_hora: datetime) RegistroHistorico
    }

    class ListaPersonalizada {
        -_nome: String
        -_midias: dict~int, Midia~
        +nome() String
        +midias() List~Midia~
        +adicionar_midia(midia: Midia) void
        +remover_midia(midia: Midia) void
        +__contains__(item: Object) bool
        +__len__() int
    }

    class RegistroHistorico {
        -_midia: Midia
        -_data_hora: datetime
        +midia() Midia
        +data_hora() datetime
    }

    %% ==========================================
    %% RELACIONAMENTOS DE HERANÇA E MIXINS
    %% ==========================================
    Midia <|-- Filme
    Midia <|-- Serie
    AvaliavelMixin <|-- Midia
    SerializavelMixin <|-- Midia
    AvaliavelMixin <|-- Episodio
    SerializavelMixin <|-- Episodio
    SerializavelMixin <|-- Temporada
    SerializavelMixin <|-- Usuario
    SerializavelMixin <|-- ListaPersonalizada
    SerializavelMixin <|-- RegistroHistorico

    %% ==========================================
    %% RELACIONAMENTOS DE ASSOCIAÇÃO
    %% ==========================================
    Serie "1" o-- "*" Temporada
    Temporada "1" *-- "*" Episodio
    Usuario "1" *-- "*" ListaPersonalizada
    Usuario "1" *-- "*" RegistroHistorico
    Usuario "1" *-- "1" ListaPersonalizada
    ListaPersonalizada "*" o-- "*" Midia
    RegistroHistorico --> Midia

    Midia --> TipoMidia
    Midia --> StatusVisualizacao
    Episodio --> StatusVisualizacao

    RotasAPI ..> MidiaSchemaCreate
    RotasAPI --> MidiaRepository
    RotasAPI --> UsuarioRepository
    MidiaRepositoryJSON ..> Midia
    MidiaRepositorySQLite ..> Midia
    UsuarioRepositoryJSON ..> Usuario

    %% ==========================================
    %% ESTILOS VISUAIS PERSONALIZADOS (classDef)
    %% ==========================================
    classDef interfaceStyle fill:#0369a1,color:#ffffff,stroke:#0284c7,stroke-width:2px;
    classDef repoStyle fill:#15803d,color:#ffffff,stroke:#16a34a,stroke-width:2px;
    classDef domainStyle fill:#4338ca,color:#ffffff,stroke:#4f46e5,stroke-width:2px;
    classDef enumStyle fill:#b45309,color:#ffffff,stroke:#d97706,stroke-width:2px;
    classDef mixinStyle fill:#6b21a8,color:#ffffff,stroke:#9333ea,stroke-width:2px;

    class RotasAPI interfaceStyle
    class MidiaSchemaCreate interfaceStyle
    class MidiaRepository repoStyle
    class MidiaRepositoryJSON repoStyle
    class MidiaRepositorySQLite repoStyle
    class UsuarioRepository repoStyle
    class UsuarioRepositoryJSON repoStyle
    class Midia domainStyle
    class Filme domainStyle
    class Serie domainStyle
    class Temporada domainStyle
    class Episodio domainStyle
    class Usuario domainStyle
    class ListaPersonalizada domainStyle
    class RegistroHistorico domainStyle
    class StatusVisualizacao enumStyle
    class TipoMidia enumStyle
    class DominioError enumStyle
    class ValorInvalidoError enumStyle
    class EstadoInvalidoError enumStyle
    class AvaliavelMixin mixinStyle
    class SerializavelMixin mixinStyle
```

