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
      'primaryBorderColor': '#475569',
      'lineColor': '#94a3b8',
      'tertiaryColor': 'transparent',
      'tertiaryBorderColor': '#64748b',
      'tertiaryTextColor': '#94a3b8',
      'fontSize': '11px'
    },
    'themeCSS': 'g.classGroup text { font-size: 11px !important; } .nodeLabel { padding: 8px !important; }'
  }
}%%
classDiagram
    direction LR

    namespace Camada_API {
        class RotasAPI {
            <<Router - FastAPI>>
            +criar_midia()
            +listar_midias()
            +criar_usuario()
            +obter_relatorios()
        }
        class MidiaSchemaCreate {
            <<Pydantic Schema>>
            +titulo: str
            +tipo: str
            +duracao_minutos: int
        }
    }

    namespace Camada_Core {
        class EventDispatcher {
            <<Singleton>>
            -_instancia: EventDispatcher
            -_observadores: List~Observer~
            +inscrever(observador)
            +desinscrever(observador)
            +emitir(nome_evento, dados)
        }
        class Observer {
            <<Interface - ABC>>
            +notificar(evento)*
        }
        class ConsoleLogger {
            +notificar(evento)
        }
        class FileLogger {
            -caminho: Path
            +notificar(evento)
        }
    }

    namespace Camada_Repositorios {
        class MidiaRepository {
            <<Interface - ABC>>
            +salvar_todos(midias)*
            +carregar_todos()*
        }
        class MidiaRepositoryJSON {
            +salvar_todos(midias)
            +carregar_todos()
        }
        class MidiaRepositorySQLite {
            +salvar_todos(midias)
            +carregar_todos()
        }
        class UsuarioRepositoryJSON {
            +salvar(usuario)
            +carregar(id_usuario)
        }
    }

    namespace Camada_Dominio {
        class AvaliavelMixin {
            <<Mixin>>
            #_nota: float
            +avaliar(valor)
        }
        class SerializavelMixin {
            <<Mixin>>
            +to_dict() dict
            +from_dict(dados)
        }
        class Midia {
            <<Abstract>>
            #id: int
            #titulo: str
            #tipo: TipoMidia
            #status: StatusVisualizacao
            #data_conclusao: datetime
            +duracao_minutos() int
            +alterar_status(novo_status)
        }
        class Filme {
            -duracao_minutos: int
        }
        class Serie {
            -temporadas: List~Temporada~
            +duracao_minutos() int
            +atualizar_status()
            +adicionar_temporada()
        }
        class Temporada {
            -numero: int
            -episodios: List~Episodio~
            +adicionar_episodio()
            +calcular_nota_media()
        }
        class Episodio {
            -id: int
            -titulo: str
            -duracao_minutos: int
            +alterar_status()
        }
        class Usuario {
            -id_usuario: str
            -nome: str
            -email: str
            -limite_listas: int
            +criar_lista()
            +adicionar_favorito()
            +registrar_historico()
        }
        class ListaPersonalizada {
            -nome: str
            -midias: dict
            +adicionar_midia()
            +remover_midia()
        }
        class RegistroHistorico {
            -data_hora: datetime
        }
    }

    namespace Tipos_Auxiliares {
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
    }

    %% Herança de Mixins e Domínio
    AvaliavelMixin <|-- Midia
    SerializavelMixin <|-- Midia
    SerializavelMixin <|-- Usuario
    SerializavelMixin <|-- ListaPersonalizada
    SerializavelMixin <|-- RegistroHistorico

    Midia <|-- Filme
    Midia <|-- Serie

    %% Composições e Agregações
    Serie "1" o-- "*" Temporada
    Temporada "1" *-- "*" Episodio
    Usuario "1" *-- "*" ListaPersonalizada
    Usuario "1" *-- "*" RegistroHistorico
    ListaPersonalizada "*" o-- "*" Midia
    RegistroHistorico --> Midia

    %% Uso de Enums
    Midia --> TipoMidia
    Midia --> StatusVisualizacao

    %% Estrutura do Observer
    Observer <|-- ConsoleLogger
    Observer <|-- FileLogger
    EventDispatcher "1" o-- "*" Observer
    Midia ..> EventDispatcher : emite eventos
    Usuario ..> EventDispatcher : emite eventos

    %% Persistência e API
    MidiaRepository <|-- MidiaRepositoryJSON
    MidiaRepository <|-- MidiaRepositorySQLite
    RotasAPI ..> MidiaSchemaCreate
    RotasAPI --> MidiaRepository
    MidiaRepositorySQLite ..> Midia
```
