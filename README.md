# EA LANGUAGE HUB

**Aprenda. Traduza. Comunique.**

Aplicação Flutter educativa, gratuita na versão inicial e concebida para funcionar **offline-first** em Android, iOS e Web/PWA. Não usa OpenAI, ChatGPT API, Gemini, Claude, Azure AI, AWS AI, Firebase, subscrições ou endpoints pagos.

> **Estado desta entrega:** projecto-fonte e automação de builds preparados. O ambiente onde estes ficheiros foram gerados não tem Flutter/Dart instalado; por isso não foi possível compilar, executar testes Flutter nem anexar APK/AAB/IPA. O workflow de GitHub Actions e o script de preparação fazem essa etapa num ambiente com Flutter.

## O que está incluído

- Interface Material 3 responsiva, modo claro/escuro, navegação inferior em dispositivos pequenos e `NavigationRail` em ecrãs largos.
- Entrada, registo local, sessão offline e recuperação através de um código apresentado no registo.
- Perfis locais, fotografia guardada em Hive, XP, medalhas, sequência, meta diária e ranking do dispositivo.
- 9 idiomas (Inglês, Português, Francês, Espanhol, Alemão, Italiano, Árabe, Chinês e Suaíli), níveis A1–C2 e 3 lições por combinação — **162 lições curtas geradas no dispositivo**.
- Exercícios de escolha múltipla, verdadeiro/falso, completar, correspondência e escuta com TTS.
- Dicionário local com pesquisa instantânea, tradução, pronúncia de referência, classe, definição, sinónimos, antónimos, exemplos e favoritos.
- Tradutor offline de texto, histórico/favoritos, copiar/partilhar, importação de TXT/MD/CSV (limite de 1 MB), OCR de imagem no Android/iOS e ditado.
- 8 diálogos guiados pré-programados: hotel, hospital, restaurante, escola, universidade, aeroporto, negócios e entrevista.
- Certificados PDF locais, emitidos depois de pelo menos 2 lições do nível e média igual ou superior a 70%.
- Painel administrativo local: estatísticas, activar idiomas, activar/desactivar contas locais e criar lições de vocabulário.
- Exportação JSON de uma cópia dos dados de perfil, sem hashes de autenticação.
- SQLite no Android/iOS; Hive para estado/preferências e persistência equivalente no Web.
- Logótipo vectorial claro/escuro, marca compacta, ecrã de abertura e ícones PWA.
- Testes unitários, widget, integração e workflow GitHub para Web, APK, AAB e build iOS sem assinatura.

## Limites funcionais deliberados

Esta versão não usa IA nem um serviço de tradução. O tradutor é um **dicionário/fraseário offline pequeno**, com termos e frases de exemplo em `lib/data/seed_data.dart`; texto fora do vocabulário permanece visível no original e é assinalado. Não é tradução automática geral.

- O conteúdo de aprendizagem é material demonstrativo curto, não um curso editorial completo para certificação académica.
- Changana, Ronga, Macua, Sena, Ndau, Tsonga, Zulu e outras línguas africanas estão previstas na expansão, mas não estão no catálogo funcional deste MVP.
- OCR usa Google ML Kit local no Android/iOS e o reconhecedor Latin nesta versão. O modo OCR não está disponível na Web/PWA.
- Voz para texto usa `speech_to_text` na app móvel; a PWA desactiva o reconhecimento de voz para não depender do serviço do navegador. O reconhecimento offline varia por dispositivo e modelo de voz. Texto para voz usa as vozes do sistema/navegador disponíveis localmente.
- Documentos suportados no importador: TXT, MD e CSV; PDF/Word não são extraídos nesta versão.
- O painel administrativo e as contas só existem no dispositivo; não há sincronização multiutilizador nem gestão remota.
- O backup é exportado/partilhado como JSON; não existe ainda importação/restauro guiado.

## Tecnologias

| Área | Implementação |
|---|---|
| Aplicação / Web / PWA | Flutter / Flutter Web |
| Estado | Provider |
| Persistência Android/iOS | SQLite via `sqflite` |
| Preferências e Web | Hive / Hive Flutter |
| Hash local | PBKDF2-HMAC-SHA256 com salt aleatório (60 000 iterações) |
| OCR móvel | Google ML Kit Text Recognition |
| Texto para voz | `flutter_tts` |
| Voz para texto | `speech_to_text` |
| PDF | `pdf` + `printing` |
| Imagens e documentos | `image_picker`, `file_picker` |

### Persistência Web

O plugin `sqflite` não tem implementação Web oficial neste projecto. A aplicação selecciona Hive no Flutter Web para que a PWA não tente abrir um SQLite nativo. Android/iOS usam SQLite como base de dados local; ambos os caminhos mantêm os dados no dispositivo e não requerem um backend.

## Instalação e desenvolvimento

### Requisitos

- Flutter stable 3.32 ou superior (Dart 3.8+).
- Android Studio/Android SDK para Android; Xcode numa máquina macOS para iOS.
- Python 3 para o script de configuração dos wrappers.

```bash
git clone <URL-DO-REPOSITORIO>
cd EA_LANGUAGE_HUB
bash scripts/bootstrap_platforms.sh
```

O script cria os wrappers `android/`, `ios/` e `web/`, preserva os ficheiros PWA da marca, configura permissões da câmara/microfone, define o deployment target iOS necessário ao ML Kit e resolve dependências. O `pubspec.lock` será criado no primeiro `flutter pub get`; versione esse ficheiro depois de rever as versões resolvidas para builds reproduzíveis.

```bash
flutter run -d chrome
flutter run -d android
flutter test
flutter test --coverage
flutter analyze
```

No iOS, use Xcode/macOS para configurar assinatura e executar. As permissões de câmara, biblioteca de fotografias, microfone e reconhecimento de voz são declaradas com descrições para consentimento do utilizador.

## Builds

```bash
# Web e PWA
flutter build web --release
# saída: build/web/

# Android
flutter build apk --release
flutter build appbundle --release
# saídas: build/app/outputs/flutter-apk/ e build/app/outputs/bundle/release/

# iOS (macOS; a distribuição final exige assinatura Apple)
flutter build ios --release --no-codesign
```

O workflow `.github/workflows/build.yml` executa análise, testes e builds Android/Web; num runner macOS compila iOS sem assinatura. Os ficheiros de build são publicados como artefactos da execução, não como binários assinados para as lojas.

### Instalar a PWA

Publique `build/web/` num domínio HTTPS (ou use `localhost` em desenvolvimento), abra no navegador e escolha **Instalar aplicação / Adicionar ao ecrã inicial**. `web/sw.js` guarda no cache recursos da mesma origem à medida que são carregados e apresenta a shell local quando a ligação falha.

A **primeira visita** precisa de transferir a aplicação do anfitrião Web. Depois de carregada, a cache permite abrir a PWA offline. Ao publicar uma nova versão, aumente `CACHE_NAME` em `web/sw.js` para renovar a cache nos dispositivos.

## Testes e cobertura

```bash
flutter test
flutter test --coverage
flutter test integration_test/app_smoke_test.dart -d <ID-DO-DISPOSITIVO>
```

Os testes incluídos cobrem validação de formulários, hashing, tradutor offline, logótipo e um smoke test de integração. **A meta de 80% de cobertura ainda não foi medida nem certificada**; a percentagem deve ser verificada no `coverage/lcov.info` e ampliada antes de uma publicação de produção.

## Segurança e privacidade

- Palavra-passe e código de recuperação são derivados com PBKDF2-HMAC-SHA256 e salt aleatório; o código de recuperação é mostrado apenas uma vez no registo.
- Consultas SQL usam parâmetros e nomes de tabela fixos; campos de formulário são validados.
- Exportação JSON omite hashes e salts de autenticação.
- Dados de perfil, histórico e progresso ficam no armazenamento local. Não há criptografia da base SQLite/Hive nem recuperação remota; proteger o acesso físico ao dispositivo e guardar a cópia exportada com cuidado.
- O painel administrativo é uma ferramenta local de conveniência, **não** uma fronteira de segurança.
- Para produção multiutilizador, distribuição em lojas ou dados sensíveis, é necessária revisão independente de segurança, acessibilidade, licenças, permissões, política de privacidade e testes nos dispositivos-alvo.

## Estrutura

```text
lib/
  app.dart, main.dart
  core/          constantes, SQLite, armazenamento Hive e validação/hash
  data/          vocabulário e conteúdo offline
  models/        utilizador, idioma, lição, léxico e tradução
  providers/     autenticação, progresso e preferências
  repositories/  acesso local a utilizadores e progresso
  routes/        nomes de rotas
  screens/       entrada, aprendizagem, tradutor, dicionário, perfil e admin
  services/      autenticação, tradução local, OCR, voz e certificados
  theme/         temas claro/escuro
  widgets/       marca, cartões e títulos acessíveis
assets/          logótipos SVG, ícones, fontes e pastas de conteúdo
admin/           descrição do painel local
database/        esquema SQL de referência
integration_test/ smoke test de dispositivo
test/            testes unitários e widget
web/             index, manifest, service worker e ícones PWA
scripts/         criação/configuração dos wrappers Android/iOS/Web
.github/         builds e testes automatizados
```

## Lojas móveis

Este repositório prepara a estrutura de compilação, mas não inclui um APK/AAB assinado, certificado Apple, screenshots, ficha de loja nem publicação. Antes de Google Play ou Apple App Store, configurar IDs de pacote definitivos, certificados/assinatura, ícones nativos, declarações de privacidade e permissões, política de conteúdo, testes nos dispositivos e requisitos actuais de cada loja. O workflow gera artefactos não assinados automaticamente quando executado no GitHub.

## Licença

Código e conteúdo deste protótipo: MIT (ver `LICENSE`). Fontes DejaVu incluídas para PDF, com licença em `assets/fonts/DejaVu-LICENSE.txt`.
