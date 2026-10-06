# CI/CD e releases

O repositório possui três workflows principais em GitHub Actions.

## CI (Blindado e Seguro)

Arquivo: `.github/workflows/ci.yml`

Responsabilidades:

- validar formatação com `gofmt`
- rodar `golangci-lint`
- rodar `go test ./...`
- SAST com Semgrep (OWASP Top 10 + security-audit)
- scan de vulnerabilidades da imagem Docker com Trivy (CRITICAL, HIGH)
- validar os arquivos Compose
- buildar as três imagens Docker

Blindagens implementadas:

- `permissions: contents: read` (menor privilégio)
- `concurrency: cancel-in-progress: true` (evitar execuções redundantes)
- `timeout-minutes: 20` em todos os jobs
- cache nativo de dependências Go (`cache: true`)
- Actions fixadas em versões estáveis (NUNCA `@master`)

## Publish Docker Images and Release

Arquivo: `.github/workflows/publish-docker.yml`

Responsabilidades:

- calcular a próxima versão a partir dos labels do PR
- validar código e Compose antes da release
- buildar imagens locais para scan
- rodar Trivy antes do push das imagens
- publicar imagens no GHCR
- criar uma release no GitHub

## Release Please & Publish Multi-Arch Container

Arquivo: `.github/workflows/release-package.yml`

Responsabilidades:

- analisar Conventional Commits e manter o Release PR automaticamente (Google Release Please v4)
- gerar `CHANGELOG.md` e tag SemVer automática ao merge do Release PR
- compilar imagem Docker para `linux/amd64` e `linux/arm64` via Docker Buildx + QEMU
- publicar imagem multi-arch no GHCR com tags `latest`, versão completa e `major.minor`

## Conventional Commits para Release

O Google Release Please interpreta as mensagens de commit seguindo o padrão:

- `feat:` → incrementa **MINOR** (ex: 1.0.0 → 1.1.0)
- `fix:` → incrementa **PATCH** (ex: 1.0.0 → 1.0.1)
- `feat!:` ou `BREAKING CHANGE:` → incrementa **MAJOR** (ex: 1.0.0 → 2.0.0)

## Nomes das imagens publicadas

- `ghcr.io/<owner>/gotodolist:<versao>`
- `ghcr.io/<owner>/gotodolist:latest`