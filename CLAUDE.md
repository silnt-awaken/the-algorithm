# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is Twitter's open-sourced recommendation algorithm codebase, a large-scale distributed system that powers Twitter's timeline and content recommendation features. The architecture follows a microservices pattern with sophisticated ML pipelines for candidate generation, ranking, and mixing.

## Build System

The primary build system is **Bazel**. Most components define their build targets in `BUILD` or `BUILD.bazel` files.

### Common Build Commands

```bash
# Build a Scala service
bazel build //path/to/service:binary

# Run tests for a component
bazel test //path/to/service:test

# Build with specific JVM options
bazel build --jvmopt="-Xmx4G" //target

# For Rust components (Navi)
cd navi/navi
cargo build --features=tf  # TensorFlow support
cargo test

# For Python components (TWML)
cd twml
pip install -e .
```

## Architecture Overview

The system follows a pipeline architecture for recommendations:

1. **Candidate Generation** (CR-Mixer) - Retrieves potential tweets from various sources
2. **Feature Hydration** - Enriches candidates with user/tweet features
3. **Ranking** - Scores candidates using ML models (Light Ranker → Heavy Ranker)
4. **Filtering** - Applies business rules and safety filters
5. **Mixing** (Home-Mixer) - Combines and orders final timeline

### Key Components

- **Product Mixer**: Framework for building recommendation pipelines (`product-mixer/`)
- **Home Mixer**: Timeline construction service (`home-mixer/`)
- **CR-Mixer**: Candidate retrieval service (`cr-mixer/`)
- **SimClusters**: Community-based embeddings (`src/scala/com/twitter/simclusters_v2/`)
- **Tweetypie**: Tweet CRUD service (`tweetypie/`)
- **Navi**: High-performance ML serving in Rust (`navi/`)
- **Earlybird**: Real-time search index (`src/java/com/twitter/search/`)

## Code Organization

### Language Distribution
- **Scala**: Primary language for services and data processing
- **Java**: Search infrastructure and some core components
- **Rust**: ML model serving (Navi)
- **Python**: ML training and data science (TWML)
- **Thrift**: Service definitions and schemas

### Directory Structure
```
<service-name>/
├── server/           # Main service implementation
├── client/           # Client libraries
├── thrift/           # Service-specific Thrift definitions
├── common/           # Shared utilities
└── config/           # Configuration files (decider.yml)

src/
├── scala/com/twitter/    # Scala libraries and components
├── java/com/twitter/     # Java components
├── python/twitter/       # Python ML code
└── thrift/com/twitter/   # Shared Thrift definitions
```

## Testing

Tests use ScalaTest with JUnit integration. Test files follow `*Spec.scala` naming convention.

```bash
# Run all tests for a service
bazel test //service-name/server/...

# Run specific test
bazel test //service-name/server/src/test/scala/com/twitter/service:TestSpec

# Run tests with specific tags
bazel test --test_tag_filters=bazel-compatible //...
```

## Development Patterns

### Service Implementation Pattern
Services typically follow this structure:
- **Handlers**: Request handling logic
- **Repositories**: Data access layer
- **Stores**: Storage abstraction (Manhattan, Memcache)
- **Hydrators**: Feature enrichment
- **Filters**: Business logic filtering
- **Transformers**: Data transformation

### Configuration Management
- Feature flags: `config/decider.yml`
- Service configuration: Typically in `server/Main.scala` or module definitions
- Logging: `resources/logback.xml`

### Thrift Service Pattern
```scala
// Service definition in thrift/
// Implementation in server/
// Client in client/
```

## Key Technologies

- **Storage**: Manhattan (key-value), Twemcache (caching)
- **Stream Processing**: Storm/Heron topologies
- **Batch Processing**: Scalding on Hadoop
- **ML Frameworks**: TensorFlow, PyTorch, ONNX
- **RPC**: Finagle with Thrift
- **Feature Store**: Feature hydration from various stores

## Important Files to Review

- `/README.md` - Main project overview
- `/product-mixer/README.md` - Pipeline framework documentation
- `/home-mixer/README.md` - Timeline algorithm details
- `/navi/README.md` - ML serving architecture
- `/RETRIEVAL_SIGNALS.md` - User behavior signals reference

## Working with ML Models

### Navi (Rust ML Server)
```bash
cd navi/navi
cargo build --features=tf,onnx  # Build with multiple backends
cargo test
```

### TWML (Python ML Framework)
```bash
cd twml
python setup.py develop
```

## Common Development Tasks

### Adding a New Candidate Source
1. Implement in `cr-mixer/server/src/main/scala/com/twitter/cr_mixer/source/`
2. Add to candidate pipeline in `cr-mixer/server/src/main/scala/com/twitter/cr_mixer/`
3. Update BUILD file with dependencies

### Modifying Ranking Logic
1. Light Ranker: `src/python/twitter/deepbird/projects/timelines/scripts/models/earlybird/`
2. Heavy Ranker: `src/scala/com/twitter/home_mixer/functional_component/scorer/`

### Adding Feature Hydration
1. Implement hydrator in appropriate service
2. Add to pipeline configuration in Product Mixer component
3. Update feature schema if needed