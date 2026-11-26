<div align="center">

<img src="Resources/AppIcon/appicon.svg" width="96" alt="CARGAS BRASIL" />

# CARGAS BRASIL

**App de gestão de frota para caminhoneiros autônomos e pequenas transportadoras.**

Acompanhe caminhão, carretas, cargas disponíveis e fretes realizados, tudo em um só lugar.

![.NET](https://img.shields.io/badge/.NET-10-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![MAUI](https://img.shields.io/badge/.NET_MAUI-multiplataforma-512BD4?style=for-the-badge&logo=dotnet&logoColor=white)
![C#](https://img.shields.io/badge/C%23-239120?style=for-the-badge&logo=csharp&logoColor=white)
![MVVM](https://img.shields.io/badge/MVVM-CommunityToolkit-2962FF?style=for-the-badge)
![Status](https://img.shields.io/badge/status-protótipo-FF5252?style=for-the-badge)

![Android](https://img.shields.io/badge/Android-3DDC84?style=flat-square&logo=android&logoColor=white)
![iOS](https://img.shields.io/badge/iOS-000000?style=flat-square&logo=apple&logoColor=white)
![macOS](https://img.shields.io/badge/MacCatalyst-000000?style=flat-square&logo=apple&logoColor=white)
![Windows](https://img.shields.io/badge/Windows-0078D4?style=flat-square&logo=windows&logoColor=white)

[Telas](#-telas) •
[Funcionalidades](#-funcionalidades) •
[Arquitetura](#%EF%B8%8F-arquitetura) •
[Como rodar](#%EF%B8%8F-como-rodar) •
[Roadmap](#%EF%B8%8F-roadmap)

</div>

---

> [!NOTE]
> **Base técnica pronta.** Navegação, arquitetura, telas e camada de serviços estão funcionais com **dados simulados** (com delay de rede). A integração com a API real é o próximo passo. Veja o [Roadmap](#%EF%B8%8F-roadmap).

---

## 📱 Telas

| Login | Início | Caminhão | Cargas | Fretes | Perfil |
|:---:|:---:|:---:|:---:|:---:|:---:|
| <img src="docs/screenshots/login.png" width="160"/> | <img src="docs/screenshots/home.png" width="160"/> | <img src="docs/screenshots/truck.png" width="160"/> | <img src="docs/screenshots/loads.png" width="160"/> | <img src="docs/screenshots/freights.png" width="160"/> | <img src="docs/screenshots/profile.png" width="160"/> |

<details>
<summary><b>🌙 Tema escuro</b></summary>

| Início | Caminhão | Cargas | Perfil |
|:---:|:---:|:---:|:---:|
| <img src="docs/screenshots/home-dark.png" width="180"/> | <img src="docs/screenshots/truck-dark.png" width="180"/> | <img src="docs/screenshots/loads-dark.png" width="180"/> | <img src="docs/screenshots/profile-dark.png" width="180"/> |

</details>

<details>
<summary><b>🖥️ Versão desktop (Windows)</b></summary>

<img src="docs/screenshots/desktop.png" alt="CargaBR no Windows" />

</details>

---

## ✨ Funcionalidades

| | Tela | O que faz |
|:---:|---|---|
| 🔐 | **Login / Cadastro** | Validação de formulário, loading, mensagens de erro e sessão persistida |
| 🏠 | **Início** | Resumo de ganhos do mês, entregas e atalhos rápidos |
| 🚚 | **Meu Caminhão** | Dados do veículo, lista de carretas e cadastro de nova carreta |
| 📦 | **Cargas Disponíveis** | Lista de cargas com origem, destino, peso, distância e aceite |
| 🧾 | **Meus Fretes** | Histórico de fretes com status (em andamento / concluído) |
| 👤 | **Perfil** | Dados do usuário, assinatura, troca de tema e logout |
| 🌗 | **Tema** | Claro, Escuro ou Sistema, salvo entre sessões |

---

## 🧭 Navegação

```mermaid
flowchart LR
    S([🚀 Splash]) --> A{Sessão salva?}
    A -->|Não| L[🔐 Login]
    L <--> R[📝 Cadastro]
    L --> T
    R --> T
    A -->|Sim| T

    subgraph T["📱 TabBar"]
        direction LR
        H[🏠 Início]
        C[🚚 Caminhão]
        G[📦 Cargas]
        F[🧾 Fretes]
        P[👤 Perfil]
    end

    P -->|Logout| L
```

---

## 🏗️ Arquitetura

MVVM com injeção de dependência. As telas dependem **apenas de interfaces**, então trocar os dados fake pela API real não exige mexer em nenhuma View ou ViewModel.

```mermaid
flowchart TB
    subgraph V["🖼️ Views (XAML)"]
        Pages["HomePage · TruckPage · LoadPage<br/>FreightPage · SettingPage · Auth/*"]
    end

    subgraph VM["🧠 ViewModels"]
        VMs["[ObservableProperty]<br/>[RelayCommand]"]
    end

    subgraph SVC["⚙️ Services/Api (interfaces)"]
        I1[[IAuthService]]
        I2[[ITruckService]]
        I3[[ILoadService]]
        I4[[IFreightService]]
    end

    subgraph IMPL["Implementações"]
        Fake["Fake/* ✅ atual"]
        Api["*ApiService 🔜 HttpClient"]
    end

    Pages -- "compiled bindings" --> VMs
    VMs --> I1 & I2 & I3 & I4
    I1 & I2 & I3 & I4 --> Fake
    I1 & I2 & I3 & I4 -.-> Api
    Api -.-> Backend[(API CargaBR)]
```

<details>
<summary><b>🗂️ Modelo de domínio</b></summary>

```mermaid
classDiagram
    class User {
        string FullName
        string Cpf
        string Email
        bool IsPremium
        DateTime? SubscriptionExpiration
    }
    class Truck {
        string Model
        string Type
        string Plate
        TruckSize Size
    }
    class Trailer {
        string Description
        string Plate
    }
    class Load {
        string OriginCity
        string DestinationCity
        decimal Value
        double WeightTons
        double DistanceKm
        LoadStatus Status
    }
    class Freight {
        string OriginCity
        string DestinationCity
        decimal Value
        DateOnly PeriodStart
        DateOnly PeriodEnd
        FreightStatus Status
    }
    class TruckSize {
        <<enum>>
        Pequeno
        Medio
        Grande
    }
    class LoadStatus {
        <<enum>>
        Disponivel
        Aceita
    }
    class FreightStatus {
        <<enum>>
        EmAndamento
        Concluido
    }
    Truck --> TruckSize
    Load --> LoadStatus
    Freight --> FreightStatus
```

</details>

<details>
<summary><b>📁 Estrutura de pastas</b></summary>

```
CargaBR/
├── App.xaml(.cs)          # Bootstrap, aplica o tema salvo
├── AppShell.xaml(.cs)     # Rotas + TabBar
├── MauiProgram.cs         # DI, fontes, CommunityToolkit
├── Models/                # Truck, Trailer, Load, Freight, User
│   └── Enums/             # TruckSize, LoadStatus, FreightStatus
├── Services/
│   ├── Api/               # Interfaces de domínio
│   │   └── Fake/          # Implementações simuladas
│   ├── Theme/             # Troca e persistência de tema
│   └── SubscriptionService.cs
├── ViewModels/            # Um por tela (+ Auth/)
├── Views/                 # Páginas XAML (+ Auth/)
├── Converters/            # Conversores de binding
└── Resources/
    ├── Styles/            # Colors.xaml, Styles.xaml
    ├── AppIcon/, Splash/  # Identidade visual
    └── Fonts/             # Open Sans + Font Awesome
```

</details>

---

## 🎨 Identidade visual

| Cor | Hex | Uso |
|:---:|---|---|
| ![](https://img.shields.io/badge/-%20%20%20%20-2962FF?style=flat-square) | `#2962FF` | Primária |
| ![](https://img.shields.io/badge/-%20%20%20%20-1C2331?style=flat-square) | `#1C2331` | Secundária, ícone e splash |
| ![](https://img.shields.io/badge/-%20%20%20%20-2E7D32?style=flat-square) | `#2E7D32` | Sucesso |
| ![](https://img.shields.io/badge/-%20%20%20%20-FF5252?style=flat-square) | `#FF5252` | Alerta |

Fontes: **Open Sans** (texto) e **Font Awesome 7** (ícones da TabBar e da interface).

---

## ▶️ Como rodar

**Pré-requisitos:** .NET 10 SDK + workloads MAUI, ou Visual Studio 2022 17.14+ com a carga de trabalho .NET MAUI.

| Plataforma | Comando |
|---|---|
| 🪟 Windows | `dotnet build CargaBR.csproj -f net10.0-windows10.0.19041.0 -t:Run` |
| 🤖 Android | `dotnet build CargaBR.csproj -f net10.0-android -t:Run` |
| 🧰 Todas | `dotnet build CargaBR.csproj` |

Ou abra `CargaBR.slnx` no Visual Studio e rode direto pela IDE.

> 💡 O login é simulado: qualquer e-mail e senha que passem na validação do formulário entram no app.

---

## 🛣️ Roadmap

- [x] Arquitetura MVVM + DI
- [x] Navegação Shell com TabBar
- [x] Autenticação simulada com sessão persistida
- [x] Tema claro/escuro/sistema
- [x] Telas de Início, Caminhão, Cargas, Fretes e Perfil
- [ ] Implementações reais dos serviços com `HttpClient`
- [ ] Autenticação JWT
- [ ] Assinatura premium ligada à API
- [ ] Publicação nas lojas

<details>
<summary><b>🔌 Como plugar a API real</b></summary>

1. Crie as implementações reais, por exemplo `TruckApiService : ITruckService`.
2. Defina a URL base em `Services/Api/ApiSettings.cs`.
3. No `MauiProgram.cs`, troque o registro fake:

```csharp
builder.Services.AddHttpClient<ITruckService, TruckApiService>(c => c.BaseAddress = new Uri(ApiSettings.BaseUrl));
```

4. Repita para `IAuthService`, `ILoadService` e `IFreightService`.
5. Busque por `// TODO` no código: cada método fake indica a chamada HTTP que deve substituí-lo.

</details>

---

<div align="center">

Feito por **[Guilherme Amorim](https://github.com/gu1Amorim)** · [LinkedIn](https://linkedin.com/in/gu1amorim)

</div>
