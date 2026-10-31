# Solução de erros no IntelliJ IDEA 2025.1 (Java 21 + Maven)

Projeto gerado pelo Spring Initializr (Spring Boot 4.0.9, Java 21).

---

## Erro 1: `release version 5 not supported`

```
java: error: release version 5 not supported

Module controledegastos SDK 21 does not support source version 1.5.
```

### Causa

O `pom.xml` e o `.idea/misc.xml` já estão corretos (Java 21). O erro ocorre quando o IntelliJ **não consegue importar a configuração do Maven** e o módulo volta ao nível padrão Java 1.5 (`release 5`), enquanto o SDK é o 21, que não compila mais Java 5.

Isso costuma acontecer quando o projeto é aberto antes do Maven terminar de sincronizar, ou quando a importação falha.

### Correção (nesta ordem)

1. **Recarregar o Maven**: aba Maven > ícone **Reload All Maven Projects**. Espere terminar sem erros.
2. **JDK do importador**: `File > Settings > Build, Execution, Deployment > Build Tools > Maven > Importing`. Defina **JDK for importer** como `21` ou `Use Project JDK`. Em `Maven > Runner`, use também o JDK 21.
3. **Nível do módulo**: `File > Project Structure > Modules > controledegastos > Sources`. Defina **Language level** como `21` (ou `Project default`). Em `Project`, SDK e Language level devem ser 21.
4. **Bytecode**: `Settings > Build, Execution, Deployment > Compiler > Java Compiler`. Defina **Target bytecode version** de `controledegastos` como `21`.
5. **Se ainda falhar**:
   - `File > Invalidate Caches > Invalidate and Restart`.
   - Ou feche o projeto, apague a pasta `.idea` (não é versionada) e reabra o `pom.xml` com **Open as Project**.

> Opcional: adicionar `<maven.compiler.release>21</maven.compiler.release>` em `<properties>` do `pom.xml` deixa a configuração mais explícita. Normalmente não é necessário, pois o parent do Spring Boot já define isso.

---

## Erro 2: `No implementation for ArtifactHandlerManager was bound`

```
Unable to create injector, see the following errors:

1) [Guice/MissingImplementation]: No implementation for ArtifactHandlerManager was bound.
   Requested by: DefaultArtifactFactory.artifactHandlerManager(...)
```

### Causa

O arquivo `.mvn/wrapper/maven-wrapper.properties` gerado pelo Initializr aponta para o **Maven 3.10.0**:

```
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.10.0/apache-maven-3.10.0-bin.zip
```

Quando o IntelliJ usa o wrapper (`Maven home path = Use Maven wrapper`), ele baixa o 3.10.0. Essa versão mudou a estrutura interna do Maven, e o IntelliJ 2025.1 não a suporta. O resultado é uma falha ao criar o injetor Guice/Plexus. É uma **incompatibilidade entre o IDE e a versão do Maven**, não um problema no código.

### Correção

**Opção A: só na IDE**

1. `File > Settings > Build, Execution, Deployment > Build Tools > Maven`.
2. Em **Maven home path**, troque `Use Maven wrapper` por `Bundled (Maven 3)` ou por uma instalação local do Maven 3.9.x (ex.: `D:\Dev\Tools\Java\apache-maven-3.9.12`).
3. Em `Runner`, confirme o JDK 21.
4. Clique em **Reload All Maven Projects**.

**Opção B: corrigir o wrapper (vale para quem clonar o projeto)**

Edite `.mvn/wrapper/maven-wrapper.properties`:

```
distributionUrl=https://repo.maven.apache.org/maven2/org/apache/maven/apache-maven/3.9.12/apache-maven-3.9.12-bin.zip
```

Depois recarregue o Maven. Se o IntelliJ continuar usando o 3.10.0, apague a pasta `C:\Users\<usuario>\.m2\wrapper\dists\apache-maven-3.10.0*`.

**Recomendação:** aplicar as duas opções. A B evita o mesmo erro para outras pessoas que abrirem o projeto.

---

## Verificação

No terminal, na raiz do projeto:

```
.\mvnw.cmd -v
```

A saída deve mostrar **Maven 3.9.x** e **Java 21**. Se mostrar outra versão do Java, ajuste o `JAVA_HOME`.
