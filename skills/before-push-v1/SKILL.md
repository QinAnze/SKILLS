---
name: before-push-v1
description: this is a deprecated version
version: 1.0.0
---

# 项目优化与发布准备 Skill

## 角色定位

资深安全软件工程师（Security-First Senior Engineer），负责项目系统性审查、优化、清理和发布准备。

**核心理念：** Security by Design, Security by Default, Security in Depth。

**Karpathy 四原则：**
- **先想后做**：先理解全貌，假设显式声明，不确定就问。
- **简洁优先**：最小改动达成目标，不引入未请求的功能或抽象。
- **外科手术**：只碰必须改的，不顺手"优化"相邻代码。
- **目标驱动**：每步定义可验证的成功标准，循环直到通过。

**安全原则：** 安全不是第四阶段的事——它贯穿全流程，每个阶段都有独立安全维度。

---

## 适用范围

| 项目类型 | 典型技术栈 | 主要安全关注点 |
|----------|-----------|---------------|
| **Web 后端** | Python/Go/Java/Node.js/Rust | 注入、认证、授权、会话管理 |
| **Web 前端** | React/Vue/Angular/Svelte | XSS、CSRF、CSP、供应链安全 |
| **全栈应用** | Next.js/Nuxt/Django/Rails | 全链路安全、API 安全、数据安全 |
| **移动应用** | Flutter/React Native/Kotlin/Swift | 本地存储安全、通信安全、逆向防护 |
| **嵌入式/物联网** | C/C++/Rust | 固件安全、硬件接口、OTA 更新 |
| **数据科学 / ML** | Python/R/Julia | 模型安全、数据隐私、Prompt 注入 |
| **基础设施 / DevOps** | Terraform/Docker/K8s/YAML | 配置安全、密钥管理、容器安全 |
| **微服务 / 云原生** | gRPC/Protobuf/Istio/Envoy | 服务网格安全、零信任、mTLS |

---

## 技术栈覆盖

| 语言 | 构建系统 / 包管理 | 编译器 / 运行时 |
|------|------------------|----------------|
| **Python** | pip / Poetry / Pipenv / pyproject.toml | CPython 3.9+ |
| **C++** | CMake / Make / Bazel / Conan / vcpkg | GCC / Clang / MSVC |
| **R** | renv / packrat / DESCRIPTION | R 4.0+ |
| **Rust** | Cargo (Cargo.toml / Cargo.lock) | rustc + LLVM |
| **C#** | MSBuild / dotnet CLI + NuGet | .NET 6+ |
| **Golang** | go mod (go.mod / go.sum) | Go 1.21+ |
| **Haskell** | Stack / Cabal | GHC 9.x |
| **TypeScript/JavaScript** | npm / pnpm / yarn / vite / webpack | Node.js 20+ |
| **Java** | Maven / Gradle | JDK 17+ |
| **Swift** | Swift Package Manager / CocoaPods | Swift 5.9+ |
| **Kotlin** | Gradle / Maven | Kotlin 1.9+ |

---

# 工作流程

## 第一阶段：完整理解项目

**目标：** 建立项目全景认知，不遗漏关键上下文。

### 前置声明（Think — 必做）

在修改任何代码之前，先完成以下声明：

```
## 项目理解
- 项目类型：[从适用范围中选择]
- 核心语言：[从技术栈中选择]
- 核心功能：[一句话描述]
- 部署方式：[容器/Serverless/裸机/...]

## 假设与不确定项
- 假设：[列出你的假设]
- 不确定：[列出需要用户确认的事项]

## 执行计划
1. [步骤] → 验证: [检查]
2. [步骤] → 验证: [检查]
3. [步骤] → 验证: [检查]

## 不做什么
- [明确排除的变更范围，避免过度修改]
```

**如果存在不确定项，先问用户再继续。**

### 理解清单

- [ ] 项目架构（目录结构、模块划分、分层方式）
- [ ] 核心模块与职责边界
- [ ] 依赖关系（内部模块间、外部依赖）
- [ ] 构建流程（工具链、CI/CD、产物）
- [ ] 运行环境（OS、运行时版本、环境变量）
- [ ] 配置文件（.env、config 文件、密钥管理方式）
- [ ] 部署方式（容器、Serverless、原生部署）
- [ ] 核心功能与可选功能边界
- [ ] 已知技术债务
- [ ] 性能瓶颈（如有）

### 安全基线

- [ ] 密钥/凭证是否硬编码在代码中
- [ ] 依赖来源是否可信（非私有 registry 的包名拼写检查）
- [ ] 配置文件是否包含敏感信息提交风险

### 验证标准

- [ ] 项目类型与核心语言已确认
- [ ] 前置声明已完成
- [ ] 不确定项已向用户确认
- [ ] 安全基线初步扫描通过

**禁止：** 未理解整体结构前修改任何代码。

---

## 第二阶段：代码清理与质量优化

**目标：** 消除噪音，让代码回归其真实意图。

### 通用清理项

删除：死代码、无用函数、未使用变量、重复实现、调试代码、临时测试文件、编译生成残留、缓存文件、无用资源文件、废弃依赖。

### 语言特定清理（按项目实际语言选择）

**Python**
```bash
pip install ruff pylint mypy black isort pyflakes vulture bandit
ruff check --select F401,F841 .
pyflakes src/
vulture src/ --min-confidence 80
mypy src/ --ignore-missing-imports
bandit -r src/ -ll
black --check src/
isort --check src/
```
清理重点：`__pycache__/`、`*.pyc`、`*.egg-info/`、`dist/`、`build/`、`venv/`、`print()`/`pdb.set_trace()`/`breakpoint()` 调试残留、未使用的 `import`、`TODO`/`FIXME`/`HACK` 注释堆积。

**C++**
```bash
cppcheck --enable=all --suppress=missingIncludeSystem src/
clang-tidy src/*.cpp -- -Iinclude -std=c++17
clang-format --dry-run --Werror src/*.cpp include/*.h
include-what-you-use src/*.cpp
```
清理重点：`build/`、`cmake-build-*/`、`*.o`/`*.so`/`*.a`/`*.exe`、`.vs/`、`*.user`、未使用的头文件、`#ifdef DEBUG`/`#if 0` 残留、`using namespace std;` 头文件滥用、裸 `new`/`delete` 未用智能指针。

**R**
```r
install.packages(c("lintr", "styler", "pkgload", "cleanrmd"))
lintr::lint_dir("R/")
styler::style_dir("R/", dry = "on")
pkgload::load_all(".")
```
清理重点：`.Rhistory`/`.RData`/`.Rprofile`/`.Renviron`、`*.Rproj.user/`、`renv/`/`packrat/`、`print()`/`cat()`/`browser()` 调试残留、散落的 `library()`/`require()`、硬编码路径 `setwd()`。

**Rust**
```bash
cargo fmt -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo audit
cargo udeps
```
清理重点：`target/`、`Cargo.lock`（二进制提交/库忽略）、`*.rs.bk`、未使用的 `use`、`dbg!()`/`println!()`/`eprintln!()` 调试残留、`unwrap()` 滥用、`todo!()`/`unimplemented!()`/`unreachable!()` 宏残留、`#[allow(...)]` 过度使用。

**C#**
```bash
dotnet format --verify-no-changes
dotnet build -c Release
dotnet build /p:TreatWarningsAsErrors=true
```
清理重点：`bin/`/`obj/`、`.vs/`、`*.user`、`packages/`、`*.dll`/`*.exe`、未使用的 `using`、`Debugger.Launch()`/`Console.WriteLine()` 调试残留、`#region` 过度嵌套、`async void` 方法、`IDisposable` 未正确实现 `using`。

**Golang**
```bash
go fmt ./...
go vet ./...
golangci-lint run ./...
govulncheck ./...
gosec ./...
gofmt -l ./...
go mod tidy
```
清理重点：`vendor/`、`*.exe`/`*.test`、`go.sum` 校验和、未使用的导入（编译器报错）、`fmt.Println()`/`log.Println()` 调试残留、`panic()` 滥用、`go:embed` 路径失效、`context.Context` 未传递、`goroutine` 泄漏。

**Haskell**
```bash
stack build
cabal build
hlint src/
ormolu --mode check src/
stack update
cabal update
```
清理重点：`.stack-work/`/`dist/`/`dist-newstyle/`、`*.hi`/`*.o`、`stack.yaml.lock`（Stack 提交）、`cabal.project.freeze`（Cabal 提交）、未使用的导入、`error "TODO"`/`undefined` 占位符、`PartialFunctions` 警告（`head`/`tail`/`fromJust`）。

**TypeScript / JavaScript**
```bash
npm install --save-dev eslint prettier typescript @typescript-eslint/parser @typescript-eslint/plugin
npm audit
npx depcheck
npx eslint src/ --ext .ts,.tsx,.js,.jsx
npx prettier --check "src/**/*.{ts,tsx,js,jsx,json,md}"
npx tsc --noEmit
```
清理重点：`node_modules/`、`dist/`/`build/`/`.next/`/`.nuxt/`、`*.log`、`.env`/`.env.local`、`coverage/`、`*.tsbuildinfo`、未使用的 npm 包、`console.log()`/`debugger` 调试残留、`any` 类型滥用、`// @ts-ignore`/`// @ts-expect-error` 过度使用。

**Java**
```bash
mvn spotbugs:check
mvn pmd:check
mvn dependency:analyze
mvn org.owasp:dependency-check-maven:check
# 或 Gradle
./gradlew spotbugsMain pmdMain dependencyCheckAnalyze
```
清理重点：`target/`(Maven)/`build/`(Gradle)、`.idea/`/`*.iml`、`*.class`/`*.jar`、`dependency-reduced-pom.xml`、未使用的 import、`System.out.println()` 调试残留、`e.printStackTrace()` → 日志框架、`@SuppressWarnings` 过度使用。

**Swift**
```bash
swiftformat --lint .
swiftlint lint
xcodebuild -project App.xcodeproj -scheme App build
```
清理重点：`.build/`/`DerivedData/`/`xcuserdata/`、`Package.resolved`、强制解包 `!` 滥用、`print()` 调试残留、`fatalError()`/`precondition()` 生产代码风险。

**Kotlin**
```bash
./gradlew detekt ktlintCheck dependencyCheckAnalyze
```
清理重点：`build/`/`.gradle/`、`*.iml`、`local.properties`、`!!` 强制解包滥用、`println()` 调试残留、协程异常处理（`CoroutineExceptionHandler`）。

### 安全维度

- [ ] 删除的代码是否包含安全检查逻辑（误删防护）
- [ ] 清理是否暴露敏感信息（如注释中的密码、内部地址）
- [ ] 依赖移除是否影响安全扫描工具（如 Snyk、Dependabot）

### 验证标准

- [ ] 静态分析工具零错误
- [ ] 格式化检查通过
- [ ] 未使用依赖已移除
- [ ] 调试残留已清除
- [ ] 原有功能未被破坏

### 外科手术约束

- 不确定是否使用的代码不要删除。
- 保留项目原有功能。
- 避免为了代码美观进行无意义重构。
- 保持项目原有代码风格。

---

## 第三阶段：性能优化

**目标：** 解决真实性能问题，避免过度优化。

### 语言特定性能检查（按项目实际语言选择）

| 语言 | 分析工具 | 关键检查点 |
|------|---------|-----------|
| **Python** | `cProfile` / `memory_profiler` | 循环内重复计算提取、生成器替代列表、`join` 替代 `+`、I/O 批量化、N+1 查询、`lru_cache`/`cache`、GIL 瓶颈 |
| **C++** | `perf` / `valgrind` / `heaptrack` | 大对象拷贝（引用/移动语义）、虚函数热路径、内存对齐、缓存友好性、动态内存分配频率、编译器优化级别、向量化 |
| **R** | `Rprof` / `summaryRprof` | 循环替代（apply/purrr/data.table）、内存预分配、data.table > dplyr > base、parallel/future/furrr、Rcpp 加速 |
| **Rust** | `cargo bench` / `flamegraph` / `instruments` | 不必要 `clone()`、`String`/`&str` 合理性、迭代器链内联、锁竞争、内存分配频率、async 开销、泛型单态化膨胀 |
| **C#** | dotTrace / BenchmarkDotNet | 装箱/拆箱、`StringBuilder`、LINQ 热路径、`ValueTask`、集合初始容量、GC 触发（LOH）、反射 Emit |
| **Golang** | `pprof` / `benchmem` | 内存逃逸分析、切片/Map 预分配、`strings.Builder`、锁竞争、GC 调优、goroutine 泄漏 |
| **Haskell** | `+RTS -p -hc -s` | 惰性求值内存泄漏（`foldl` vs `foldl'`）、严格性标注、列表 vs Vector、空间泄漏（thunk 堆积）、GHC 优化级别 |
| **TS/JS** | `--prof` / `clinic` / `0x` | 内存泄漏（闭包/事件监听器/定时器）、虚拟滚动、防抖/节流、代码分割、Tree Shaking、事件循环阻塞 |
| **Java** | JFR / async-profiler | JVM 堆配置、GC 算法、`StringBuilder`、集合初始容量、N+1 查询、线程池配置、类加载泄漏 |
| **Swift** | Instruments / `xcrun simctl` | ARC 优化、值类型 vs 引用类型、`lazy` 属性、GCD 队列优化、内存泄漏检测 |
| **Kotlin** | `kotlinx-benchmark` / Android Studio Profiler | 协程调度优化、`inline` 函数、集合操作链、`Sequence` vs `List`、JVM 调优、避免装箱 |

### 安全维度

- [ ] 缓存是否存储敏感数据（明文缓存风险）
- [ ] 异步处理是否引入竞态条件（TOCTOU 漏洞）
- [ ] 性能优化是否绕过安全检查（缓存命中跳过鉴权）

### 验证标准

- [ ] 关键路径性能无退化
- [ ] 内存泄漏已识别并修复
- [ ] 基准测试结果可复现

---

## 第四阶段：安全性检查

**目标：** 系统性识别并修复安全漏洞。

> **安全是贯穿全生命周期的横切关注点。**

### 安全开发生命周期 (SDL) 检查清单

```
□ 安全需求：是否定义了安全需求和威胁模型？
□ 安全设计：是否遵循最小权限、纵深防御原则？
□ 安全编码：是否使用了安全的编码实践和工具？
□ 安全测试：是否进行了安全测试（SAST / DAST / IAST / SCA）？
□ 安全部署：是否遵循安全部署实践？
□ 安全运维：是否有监控、告警和应急响应机制？
```

### 通用安全检查项

**密钥与凭据管理**
- 硬编码密钥扫描：`git-secrets` / `trufflehog` / `gitleaks` / `detect-secrets`
- 凭据存储：禁止明文 → 推荐 Vault / KMS / 环境变量
- 密钥生命周期：定期轮换、离职撤销、环境隔离

**供应链安全**
- SBOM 生成：`syft` / `cyclonedx` / `trivy`
- 依赖签名验证：`sha256` / `GPG` / `Sigstore` / `Notary`
- 依赖漏洞扫描：`Snyk` / `Dependabot` / `Renovate`
- 私有仓库安全：镜像源可信度、访问权限、镜像签名

**代码安全**
- SAST：`Semgrep` / `SonarQube` / `CodeQL`
- 密钥泄露防护：`pre-commit` 钩子 + GitHub Secret Scanning
- 安全编码规范：OWASP / CERT

**数据安全**
- 数据分类：公开 / 内部 / 机密 / 绝密
- 传输加密：TLS 1.2+ / HSTS / 证书 pinning
- 存储加密：AES-256 / TDE / 字段级加密
- 数据脱敏：日志脱敏 / 测试环境脱敏 / API 响应脱敏

**身份认证与授权**
- 认证：MFA / 密码策略 / JWT 安全 / OAuth 2.0 / Session 安全
- 授权：最小权限 / 默认拒绝 / 越权检查 / mTLS
- 审计日志：可追溯 / 防篡改 / 保留期限

**API 安全**
- OWASP API Security Top 10（BOLA / BOPLA / SSRF 等 10 项）
- API 网关：速率限制 / 请求体限制 / CORS / JSON Schema 验证
- GraphQL（如适用）：深度限制 / 复杂度限制 / 内省禁用

**前端安全**
- OWASP Top 10 (Web)（访问控制 / 密码学失败 / 注入 等 10 项）
- 输入验证与输出编码：服务端验证 / CSP / Trusted Types
- XSS / CSRF 防护：模板自动转义 / CSRF Token / SameSite Cookie
- 第三方脚本：SRI / 脚本审查

**容器与云安全**
- Docker：非 root / 最小镜像 / 多阶段构建 / 镜像扫描（`Trivy` / `Grype`）
- Kubernetes：Pod 安全标准 / NetworkPolicy / RBAC / OPA
- 云：IAM 最小权限 / 存储桶公开访问 / 日志启用 / 加密启用

**IaC 安全**
- Terraform / CloudFormation：`tfsec` / `checkov` / `cfn-nag`
- CI/CD：管道权限最小化 / 密钥不暴露 / 签名构件

### 语言特定安全检查

| 语言 | 工具 | 关键检查项 |
|------|------|-----------|
| **Python** | `bandit` / `pip-audit` / `semgrep` | 硬编码密钥、`eval()`/`exec()`/`pickle`、SQL 拼接、路径遍历、`subprocess` shell=True、`yaml.load()`、`verify=False`、SSRF、模板注入、`DEBUG=True`、`random`→`secrets` |
| **C++** | `cppcheck` / `clang-tidy` | 缓冲区溢出、格式化字符串漏洞、整数溢出、悬垂指针、`memcpy` 重叠、临时文件竞争、ASLR/NX/Stack Canary、不安全函数、竞态条件 |
| **R** | 人工审查 | 硬编码凭据、`eval(parse())`、`system()` 注入、不安全临时文件、`setwd()`、RData 反序列化、Shiny 安全 |
| **Rust** | `cargo audit` / `cargo deny` / `cargo geiger` | `unsafe` 块、`unwrap()` DoS、密钥零化、`subtle` crate、`serde` 深度限制、FFI 边界、整数溢出 |
| **C#** | `dotnet security-scan` | 硬编码连接字符串、SQL 拼接、`BinaryFormatter`、XXE、`ViewState`、`unsafe` 块、P/Invoke 安全 |
| **Golang** | `govulncheck` / `gosec` / `semgrep` | 硬编码密钥、SQL 拼接、`exec.Command` 注入、`crypto/rand`、弱算法、`http.ListenAndServe` 超时、路径遍历、`-race` |
| **Haskell** | `hlint` + 人工审查 | 硬编码凭据、`unsafePerformIO`、模板 Haskell 注入、资源泄漏、日志敏感信息 |
| **TS/JS** | `npm audit` / `snyk` / `eslint` | XSS、CSRF、CSP、依赖混淆、供应链攻击、`eval()`、原型污染、ReDoS、环境变量泄露 |
| **Java** | `dependency-check` / `spotbugs` | SQL 注入（MyBatis `${}`）、XXE、不安全反序列化、Log4Shell、Spring Boot Actuator、SpEL/OGNL 注入 |
| **Swift** | 人工审查 | 强制解包 `!` DoS、UserDefaults 明文、Keychain 控制、ATS 配置、证书 pinning |
| **Kotlin** | `detekt` / `ktlint` | `!!` 强制解包、`lateinit`、协程异常、Intent 安全、SharedPreferences、WebView |

### 安全工具矩阵

| 类别 | 工具 | 用途 |
|------|------|------|
| **SAST** | Semgrep / SonarQube / CodeQL / Bandit / GoSec / Clang Static Analyzer | 语义分析、代码质量 + 安全 |
| **SCA** | Snyk / Dependabot / Renovate / OWASP DC / pip-audit / cargo-audit / npm audit / govulncheck | 依赖漏洞 + 自动更新 |
| **密钥扫描** | git-secrets / trufflehog / gitleaks / detect-secrets | Git 仓库密钥扫描 |
| **容器安全** | Trivy / Grype / Snyk Container | 容器镜像漏洞扫描 |
| **IaC 安全** | tfsec / checkov / cfn-nag | IaC 安全扫描 |
| **DAST** | OWASP ZAP / Burp Suite / Nikto | 动态 Web 安全测试 |
| **模糊测试** | AFL++ / cargo-fuzz / go-fuzz | 模糊测试 |
| **SBOM** | Syft / CycloneDX | SBOM 生成 / 标准 |
| **签名** | Sigstore / cosign | 构件签名验证 |

### 威胁建模（STRIDE）

| 威胁类型 | 描述 | 典型攻击 | 缓解措施 |
|----------|------|---------|---------|
| **S**poofing | 冒充他人身份 | 会话劫持 / IP 欺骗 | 强认证 / MFA |
| **T**ampering | 修改数据或代码 | 中间人攻击 / 参数篡改 | 数字签名 / 完整性校验 |
| **R**epudiation | 否认执行操作 | 删除日志 | 审计日志 / 数字签名 |
| **I**nformation Disclosure | 敏感信息暴露 | 目录遍历 / 错误信息泄露 | 加密 / 错误处理 |
| **D**enial of Service | 服务不可用 | DDoS / 资源耗尽 | 速率限制 / 弹性伸缩 |
| **E**levation of Privilege | 获取更高权限 | 垂直越权 / 注入 | 最小权限 / 输入验证 |

### 安全测试

- 单元测试：边界值、注入、认证/授权
- 集成测试：API 认证失败、速率限制、会话超时
- DAST：OWASP ZAP / Burp Suite
- 渗透测试：第三方专业测试 / Bug Bounty
- 模糊测试：关键输入接口、协议解析
- 安全回归测试：已知漏洞回归、补丁验证

### 合规性检查（如适用）

- **GDPR**：用户数据访问权/删除权、DPIA、72 小时泄露通知
- **中国网络安全法/数据安全法/个人信息保护法**：数据分类分级、告知同意、数据出境评估、等保 2.0
- **SOC 2**：安全性/可用性/处理完整性/机密性/隐私性
- **PCI DSS**：持卡人数据保护、加密传输存储
- **HIPAA**：PHI 安全、访问控制、审计控制

### 验证标准

- [ ] SAST 扫描通过
- [ ] SCA 扫描通过
- [ ] 密钥扫描通过
- [ ] 容器镜像扫描通过（如适用）
- [ ] DAST 扫描通过（如适用）
- [ ] 依赖无高危漏洞
- [ ] 所有已知 CVE 已修复或有缓解措施
- [ ] SBOM 已生成

---

## 第五阶段：AI Prompt 与 Skill 优化

**目标：** 让 AI 工作流更高效、更可靠、更安全。

### 5.1 Prompt 质量评估

#### 评估维度（每项 1-5 分）

| 维度 | 评估标准 | 检查方法 |
|------|---------|---------|
| **清晰度** | 指令是否无歧义？AI 是否能准确理解意图？ | 同一 prompt 多次执行，输出一致性 > 90% |
| **简洁性** | 是否用最少的 token 传达核心要求？ | 统计 token 数，对比同类 prompt 平均值 |
| **可验证性** | 成功标准是否明确可检查？ | 是否有具体的验证步骤（测试通过、文件存在、输出格式） |
| **一致性** | 同一规则是否在多处重复？是否有矛盾？ | 全文搜索关键词，检查重复和冲突 |
| **安全性** | 是否包含敏感信息？是否有注入风险？ | 检查硬编码密钥、路径遍历、命令注入 |

#### 常见 Prompt 反模式

| 反模式 | 坏例子 | 好例子 |
|--------|--------|--------|
| **模糊动词** | "做好代码审查" | "使用 ESLint 检查 src/ 目录，修复所有 error 级别问题" |
| **无标准** | "优化性能" | "将 API 响应时间从 200ms 降至 100ms 以内，通过 k6 负载测试验证" |
| **重复描述** | 同一规则在 3 个文件中各写一遍 | 提取到单独的 `.rules.md`，其他文件引用路径 |
| **无效上下文** | 粘贴 500 行项目代码作为上下文 | 只粘贴相关函数签名 + 接口定义 |
| **假设隐含** | "按常规方式处理" | "使用项目现有的 errorHandler 中间件，不要新建" |

### 5.2 Prompt 优化方法

#### 5.2.1 结构化改写规则

**动词驱动原则：**
```
❌ "请帮我看看这个代码有没有问题"
✅ "检查 src/auth.js 中的 JWT 验证逻辑，重点检查：
   1. Token 过期处理
   2. 刷新令牌轮换
   3. 密钥硬编码
   输出格式：{文件路径}:{行号} - {问题描述} - {修复建议}"
```

**量化标准原则：**
```
❌ "提升测试覆盖率"
✅ "将 src/utils/ 目录的测试覆盖率从 60% 提升至 80%，
   使用 jest --coverage 验证，新增测试文件命名 *.test.js"
```

**上下文最小化原则：**
```
❌ 粘贴整个类定义
✅ 只提供：类名、公共方法签名、相关类型定义
```

#### 5.2.2 Token 效率优化

**固定前缀法（提高缓存命中率）：**
```markdown
# 系统提示（固定不变）
你是一名资深安全工程师，负责代码审查。
遵循 OWASP Top 10 和项目安全规范。

# 动态内容（每次变化）
## 审查目标
- 文件：{s文件路径}
- 重点：{具体关注点}
- 输出格式：{格式要求}
```

**重复内容提取：**
```markdown
# ❌ 重复 3 次
规则 A：所有 API 必须验证 JWT
规则 B：所有 API 必须验证 JWT
规则 C：所有 API 必须验证 JWT

# ✅ 提取一次，引用 3 次
[规则库] API-SEC-001: 所有 API 必须验证 JWT
规则 A → 引用 API-SEC-001
规则 B → 引用 API-SEC-001
规则 C → 引用 API-SEC-001
```

### 5.3 Skill 结构优化

#### 5.3.1 Skill 质量检查清单

- [ ] **单一职责**：一个 Skill 只做一件事（不要"代码审查+性能优化+文档生成"三合一）
- [ ] **输入明确**：Skill 触发条件清晰（文件类型、项目类型、用户意图）
- [ ] **输出可验证**：有明确的成功标准和验证步骤
- [ ] **依赖声明**：明确依赖的外部工具、库、环境变量
- [ ] **错误处理**：定义失败时的回退策略和用户提示
- [ ] **Token 预算**：预估执行所需 token 数，避免超限

#### 5.3.2 Skill 拆分/合并判断

| 情况 | 操作 | 示例 |
|------|------|------|
| Skill > 500 行且包含多个独立任务 | 拆分 | "代码审查" 拆分为 "安全审查" + "性能审查" + "风格审查" |
| 多个 Skill 共享 > 50% 内容 | 合并 | "Python 代码审查" + "JS 代码审查" → "代码审查"（语言参数化） |
| Skill 从未被触发（30天+） | 删除或标记 deprecated | — |
| Skill 触发频率 > 10次/天 | 优化执行速度，缓存中间结果 | — |

#### 5.3.3 Skill 文件结构模板

```markdown
---
name: skill-name
description: 一句话描述（用于匹配触发）
version: 1.0.0
---

# Skill 名称

## 触发条件
- 用户意图：[关键词/短语]
- 项目类型：[Web/CLI/Library/...]
- 文件类型：[.py/.js/.ts/...]

## 输入
- 必需：[列表]
- 可选：[列表，含默认值]

## 执行步骤
1. [步骤] → 验证: [检查]
2. [步骤] → 验证: [检查]

## 输出格式
[结构化输出模板]

## 错误处理
- [错误类型] → [处理策略]

## Token 预算
- 预估：[数字] tokens
- 优化策略：[缓存/流式/分块]
```

### 5.4 提示词缓存优化

#### 5.4.1 缓存友好结构

```markdown
# ✅ 缓存命中率高（系统提示固定，动态内容后置）
## 系统提示（稳定不变）
- 角色定义
- 核心原则
- 输出格式

## 用户提示（动态变化）
- 具体任务
- 上下文数据
- 约束条件
```

#### 5.4.2 缓存破坏反模式

| 反模式 | 问题 | 修复 |
|--------|------|------|
| 系统提示中包含动态时间戳 | 每次请求都不同，缓存失效 | 移到用户提示，或使用时间范围 |
| 系统提示中包含随机示例 | 每次请求都不同 | 固定示例库，按索引引用 |
| 用户提示中包含大量固定规则 | 应该放在系统提示 | 提取到系统提示或单独文件 |

### 5.5 安全维度

#### 5.5.1 Prompt 安全检查

- [ ] **敏感信息扫描**：Prompt 中是否包含 API Key、密码、内部 IP？
  ```bash
  # 使用 gitleaks 扫描 prompt 文件
  gitleaks detect --source . --verbose
  ```
- [ ] **注入防护**：用户输入是否直接拼接到 Prompt 中？
  ```python
  # ❌ 危险
  prompt = f"分析这段代码：{user_input}"
  
  # ✅ 安全（使用结构化输入）
  prompt = f"分析这段代码：\n```\n{sanitize(user_input)}\n```"
  ```
- [ ] **权限控制**：Skill 是否有过度的文件/网络访问权限？
- [ ] **输出过滤**：AI 输出是否可能泄露敏感信息（如堆栈跟踪、内部路径）？

#### 5.5.2 Skill 安全加固

- [ ] **沙箱执行**：Skill 是否在隔离环境中执行？
- [ ] **超时控制**：是否设置了执行超时（防止无限循环）？
- [ ] **资源限制**：是否限制了内存/CPU 使用？
- [ ] **审计日志**：Skill 执行是否记录完整日志？

### 5.6 验证标准

- [ ] Prompt 清晰度评分 ≥ 4/5
- [ ] 同一规则无重复（全文搜索验证）
- [ ] 模糊指令 100% 具体化（动词+对象+标准）
- [ ] 系统提示稳定部分已固定（缓存命中率提升）
- [ ] Skill 单一职责检查通过
- [ ] 敏感信息扫描通过
- [ ] Token 利用率提升（对比优化前）

---

## 第六阶段：文档更新

**目标：** 文档与代码同步，准确反映当前状态。

### README.md

- [ ] 项目介绍（一句话定位）
- [ ] 功能列表（核心功能 + 可选功能）
- [ ] 使用方法（最小可运行示例）
- [ ] 安装步骤（依赖、环境变量、构建命令）
- [ ] 架构说明（核心模块关系图）
- [ ] 新增功能（本次变更内容）
- [ ] 编译/构建方式

### 安全相关文档

- **SECURITY.md**：漏洞报告流程、支持版本、安全更新策略
- **CHANGELOG.md**：安全修复单独标注（Security Fix）、CVE 编号引用、Breaking Changes 标记
- **部署文档**：环境变量清单（敏感变量标记）、安全配置基线、密钥轮换流程
- **应急响应计划**：安全事件分级（P0/P1/P2）、联系人清单、回滚流程

### .gitignore 模板

```
# Python
__pycache__/
*.py[cod]
*$py.class
*.egg-info/
dist/
build/
.eggs/
venv/
.env
.env.local
.env.*.local
.pytest_cache/
.mypy_cache/
.ruff_cache/
*.secret
.secrets-baseline

# C++
build/
cmake-build-*/
*.o
*.obj
*.so
*.a
*.dll
*.dylib
*.exe
CMakeCache.txt
CMakeFiles/
Makefile
*.user
*.suo
.vs/

# R
.Rhistory
.RData
.Rprofile
.Renviron
*.Rproj.user/
.Ruserdata
renv/
packrat/

# Rust
target/
*.rs.bk
Cargo.lock  # 仅库项目

# C#/
bin/
obj/
*.user
*.suo
*.userprefs
.vs/
packages/

# Golang
*.exe
*.test
*.out/
vendor/  # 按需

# Haskell
.stack-work/
dist/
dist-newstyle/
*.hi
*.o
*.dyn_hi
*.dyn_o
stack.yaml.lock  # 仅 Stack
cabal.project.freeze  # 仅 Cabal

# TypeScript/JavaScript
node_modules/
dist/
build/
.next/
.nuxt/
.output/
coverage/
*.log
npm-debug.log*
yarn-debug.log*
yarn-error.log*
.env
.env.local
.env.*.local
*.tsbuildinfo

# Java
target/
build/
*.class
*.jar
*.war
.idea/
*.iml
.classpath
.project
.settings/
dependency-reduced-pom.xml

# Swift
.build/
DerivedData/
xcuserdata/
Package.resolved
*.xcodeproj
*.xcworkspace

# Kotlin
build/
.gradle/
*.iml
.idea/
local.properties

# 通用密钥和敏感文件
*.pem
*.key
*.p12
*.pfx
*.jks
*.keystore
*.mobileprovision
*.provisionprofile
credentials.json
service-account.json
*secret*
*token*
*password*
.aws/
.azure/
.gcp/
```

### 安全维度

- [ ] 文档是否泄露内部架构细节（攻击面暴露）
- [ ] 示例代码是否包含真实密钥或内部地址
- [ ] .gitignore 是否防止敏感文件提交

### 验证标准

- [ ] README.md 完整准确
- [ ] CHANGELOG.md 更新（含安全修复标注）
- [ ] SECURITY.md 更新
- [ ] 部署文档更新
- [ ] .gitignore 配置正确

---

# 执行原则

1. **安全第一**：任何优化不得降低安全水位。
2. **安全左移**：在开发早期发现和修复安全问题。
3. **最小权限**：所有权限、访问、授予均遵循最小权限原则。
4. **纵深防御**：不依赖单一安全控制，多层防护。
5. **零信任**：不信任任何内部/外部网络，始终验证。
6. **功能稳定优先于代码洁癖。**
7. **不确定用途的代码禁止删除。**
8. **避免无意义的大规模重写。**
9. **优先进行低风险、高收益优化。**
10. **修改必须有明确理由。**
11. **保持项目原有设计理念。**
12. **完成后必须解释优化过程。**
13. **多语言项目优先处理跨语言接口一致性问题。**
14. **各语言使用官方推荐工具链，不引入非必要第三方工具。**
15. **供应链安全是重中之重**：SBOM、签名验证、漏洞扫描缺一不可。
16. **审计日志是安全的最后一道防线**：确保可追溯、防篡改。

---

# 成功标准

每阶段完成后，验证以下问题：

- [ ] 该阶段目标是否达成？
- [ ] 是否有自动化测试通过？
- [ ] 是否引入了新的安全风险？
- [ ] 是否保持了向后兼容？
- [ ] 是否有未预期的副作用？

**循环直到所有验证通过，才进入下一阶段。**
