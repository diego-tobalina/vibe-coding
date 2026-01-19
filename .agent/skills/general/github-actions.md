# GitHub Actions CI/CD

## Estructura Básica

```yaml
# .github/workflows/ci.yml
name: CI Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup
        run: echo "Setting up..."
      - name: Build
        run: echo "Building..."
      - name: Test
        run: echo "Testing..."
```

## CI para Java/Maven

```yaml
name: Java CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_USER: test
          POSTGRES_PASSWORD: test
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      
      - name: Build with Maven
        run: mvn -B compile --file pom.xml
      
      - name: Run Tests
        run: mvn -B test --file pom.xml
        env:
          SPRING_DATASOURCE_URL: jdbc:postgresql://localhost:5432/testdb
          SPRING_DATASOURCE_USERNAME: test
          SPRING_DATASOURCE_PASSWORD: test
      
      - name: Upload Coverage
        uses: codecov/codecov-action@v4
        with:
          file: target/site/jacoco/jacoco.xml
```

## CI para Node/React

```yaml
name: Node CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      
      - name: Install Dependencies
        run: npm ci
      
      - name: Lint
        run: npm run lint
      
      - name: Type Check
        run: npm run type-check
      
      - name: Test
        run: npm test -- --coverage
      
      - name: Build
        run: npm run build
```

## Workflow Completo (Build + Test + Deploy)

```yaml
name: CI/CD Pipeline

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}

jobs:
  # ===================
  # BUILD & TEST
  # ===================
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: maven
      
      - name: Build & Test
        run: mvn -B verify
      
      - name: Upload JAR
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: target/*.jar

  # ===================
  # CODE QUALITY
  # ===================
  quality:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - uses: actions/checkout@v4
      
      - name: SonarQube Scan
        uses: sonarsource/sonarqube-scan-action@v2
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
          SONAR_HOST_URL: ${{ secrets.SONAR_HOST_URL }}

  # ===================
  # DOCKER BUILD
  # ===================
  docker:
    runs-on: ubuntu-latest
    needs: build
    if: github.ref == 'refs/heads/main'
    
    permissions:
      contents: read
      packages: write
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Download JAR
        uses: actions/download-artifact@v4
        with:
          name: app-jar
          path: target/
      
      - name: Login to Registry
        uses: docker/login-action@v3
        with:
          registry: ${{ env.REGISTRY }}
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      
      - name: Build and Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: |
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:latest
            ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}

  # ===================
  # DEPLOY
  # ===================
  deploy:
    runs-on: ubuntu-latest
    needs: docker
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - name: Deploy to Production
        run: |
          echo "Deploying ${{ github.sha }} to production..."
          # kubectl set image deployment/app app=${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

## Matrices (Múltiples versiones)

```yaml
jobs:
  test:
    runs-on: ubuntu-latest
    strategy:
      matrix:
        java: [17, 21]
        os: [ubuntu-latest, windows-latest]
    
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-java@v4
        with:
          java-version: ${{ matrix.java }}
          distribution: 'temurin'
      - run: mvn test
```

## Secrets y Variables

```yaml
env:
  # Variable de entorno
  APP_ENV: production
  
steps:
  - name: Use Secret
    run: echo "Connecting to ${{ secrets.DATABASE_URL }}"
    env:
      DB_PASSWORD: ${{ secrets.DB_PASSWORD }}
```

## Caché

```yaml
# Maven
- uses: actions/setup-java@v4
  with:
    cache: maven

# npm
- uses: actions/setup-node@v4
  with:
    cache: 'npm'

# Custom cache
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      ${{ runner.os }}-maven-
```

## Triggers Avanzados

```yaml
on:
  # Push a branches específicos
  push:
    branches: [main, 'release/**']
    paths:
      - 'src/**'
      - 'pom.xml'
  
  # Pull Requests
  pull_request:
    types: [opened, synchronize, reopened]
  
  # Scheduled (cron)
  schedule:
    - cron: '0 0 * * *'  # Daily at midnight
  
  # Manual
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
        type: choice
        options:
          - staging
          - production
```

## Dependencias entre Jobs

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps: [...]
  
  test:
    needs: build
    runs-on: ubuntu-latest
    steps: [...]
  
  deploy:
    needs: [build, test]
    if: success() && github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps: [...]
```

## Notificaciones

```yaml
- name: Notify Slack on Failure
  if: failure()
  uses: slackapi/slack-github-action@v1
  with:
    channel-id: 'deployments'
    payload: |
      {
        "text": "❌ Build failed for ${{ github.repository }}"
      }
  env:
    SLACK_BOT_TOKEN: ${{ secrets.SLACK_BOT_TOKEN }}
```

## Mejores Prácticas

1. **Pin versions** - `@v4` en lugar de `@main`
2. **Usar cache** - Acelera builds significativamente
3. **Secrets** - Nunca hardcodear credenciales
4. **Matrix** - Probar múltiples versiones
5. **Artifacts** - Compartir entre jobs
6. **Timeouts** - Prevenir jobs colgados
7. **Concurrency** - Cancelar runs anteriores

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```
