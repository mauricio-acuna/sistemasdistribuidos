# 🚀 ODIN - Guía Avanzada de Sistemas Distribuidos

> **Fundamentos teóricos y patrones prácticos para construir sistemas distribuidos de clase mundial**

[![Build Status](https://github.com/odin-distributed-systems/guide/workflows/Deploy/badge.svg)](https://github.com/odin-distributed-systems/guide/actions)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Contributors](https://img.shields.io/github/contributors/odin-distributed-systems/guide.svg)](https://github.com/odin-distributed-systems/guide/graphs/contributors)

## 📚 ¿Qué es ODIN?

ODIN es una guía técnica completa diseñada para equipos que desarrollan y mantienen sistemas distribuidos complejos. Combina **teoría académica sólida** con **recetas prácticas** del mundo real.

### 🎯 Audiencia Objetivo

- **Arquitectos de Software** construyendo sistemas distribuidos
- **Ingenieros Senior** implementando patrones de concurrencia avanzados  
- **DevOps/SRE** operando infraestructura crítica
- **Equipos de Desarrollo** migrando a microservicios
- **Investigadores** explorando algoritmos distribuidos

## ✨ Características Principales

### 📖 Contenido Técnico Profundo
- **Algoritmos de Consenso**: Raft, Paxos, Multi-Paxos con implementaciones
- **Modelos de Consistencia**: Desde linearizable hasta eventual consistency
- **Teoremas Fundamentales**: CAP, PACELC, FLP con casos prácticos
- **Patrones de Concurrencia**: Lock-free, actor model, CSP avanzado

### 🛠️ Implementaciones Reales
- Código producción-ready en Java, Go, Rust
- Benchmarks de performance detallados
- Casos de estudio de sistemas reales (Kafka, Cassandra, etc.)
- Testing de propiedades con jcstress, Jepsen, TLA+

### 🔧 Recetas Operacionales
- Configuraciones battle-tested
- Playbooks de troubleshooting
- Métricas y alertas esenciales
- Patterns de deployment y rollback

### 📊 Observabilidad Moderna
- OpenTelemetry distributed tracing
- Métricas SRE (SLI/SLO/Error Budgets)
- Dashboards Grafana + Prometheus
- Chaos engineering con Litmus/Chaos Monkey

## 🗂️ Estructura del Contenido

`
📁 Fundamentos Teóricos
├── 🧠 Consenso (Raft, Paxos, PBFT)
├── 🔄 Modelos de Consistencia  
├── ⚖️ CAP y PACELC Theorem
└── 🕐 Relojes Lógicos y Tiempo

📁 Concurrencia Avanzada
├── 🚫 Algoritmos Lock-Free
├── 🎭 Actors y CSP
├── 🌊 Backpressure y Flow Control
└── 🔄 Thread Pool Patterns

📁 Observabilidad
├── 📊 Métricas y Alertas
├── 🔍 Distributed Tracing
├── 📈 SRE y Error Budgets
└── �� Chaos Engineering

📁 Testing Avanzado
├── ⚡ Concurrency Testing (jcstress)
├── 🧪 Property-Based Testing
├── 🔬 Formal Verification (TLA+)
└── 💥 Chaos y Fault Injection

📁 Patrones Operacionales  
├── 🔧 Configuraciones de Producción
├── 🚨 Troubleshooting Runbooks
├── 📦 Deployment Patterns
└── 🔄 Migration Strategies

📁 Recursos
├── 📚 Papers Académicos Fundamentales
├── 🛠️ Herramientas y Frameworks
├── 🎯 Ejercicios Prácticos
└── 📖 Glosario Técnico
`

## 🚀 Quick Start

### 1. Instalación Local

`ash
# Clonar el repositorio
git clone https://github.com/odin-distributed-systems/guide.git
cd guide

# Instalar dependencias
pip install -r requirements.txt

# Ejecutar servidor local
mkdocs serve

# Navegar a http://localhost:8000
`

### 2. Docker Quickstart

`ash
# Build y run con Docker
docker run --rm -it -p 8000:8000 -v D:\j\techODIN:/docs squidfunk/mkdocs-material

# O usando docker-compose
docker-compose up -d
`

### 3. Explorar Online

🌐 **[Visitar documentación online](https://odin-distributed-systems.github.io/guide/)**

## 🎓 Rutas de Aprendizaje

### 🟢 Nivel Intermedio: Fundamentos Sólidos
1. [Introducción a Sistemas Distribuidos](docs/index.md)
2. [Algoritmos de Consenso](docs/fundamentos/consenso.md)
3. [Modelos de Consistencia](docs/fundamentos/consistencia.md)
4. [Teorema CAP en la Práctica](docs/fundamentos/cap-pacelc.md)

### 🟡 Nivel Avanzado: Concurrencia y Performance
1. [Algoritmos Lock-Free](docs/concurrencia/lock-free.md)
2. [Patrones de Backpressure](docs/concurrencia/backpressure.md)
3. [Testing de Concurrencia](docs/pruebas/jcstress.md)
4. [Observabilidad Moderna](docs/operacion/observabilidad.md)

### 🔴 Nivel Experto: Investigación y Optimización
1. [Verificación Formal con TLA+](docs/pruebas/tla-plus.md)
2. [Chaos Engineering](docs/pruebas/chaos-engineering.md)
3. [Optimización de JVM](docs/recetas/configuraciones.md)
4. [Papers Fundamentales](docs/recursos/biblioteca.md)

## 🤝 Contribuir

### 📝 Tipos de Contribución

- **�� Contenido**: Nuevos artículos, mejoras a existentes
- **💻 Código**: Ejemplos, benchmarks, implementaciones
- **🐛 Fixes**: Correcciones de errores, typos, links rotos
- **🎨 Mejoras**: Diseño, navegación, experiencia usuario
- **📊 Datos**: Benchmarks, métricas, estudios de casos

### 🔄 Flujo de Contribución

`ash
# 1. Fork y clone
git clone https://github.com/tu-usuario/guide.git

# 2. Crear branch feature
git checkout -b feature/nuevo-contenido

# 3. Hacer cambios y commit
git add .
git commit -m "feat: añadir sección sobre PBFT"

# 4. Push y crear Pull Request
git push origin feature/nuevo-contenido
# Crear PR en GitHub
`

### 📋 Guidelines de Contenido

- **Profundidad Técnica**: Incluir implementaciones, no solo teoría
- **Ejemplos Reales**: Casos de uso del mundo real
- **Código Funcional**: Todo código debe compilar y tener tests
- **Referencias**: Citar papers y fuentes académicas
- **Claridad**: Analogías y explicaciones graduales

## 🏆 Reconocimientos

### 👥 Core Contributors

- **Autor Principal**: [tu-nombre](https://github.com/tu-usuario)
- **Reviewers Técnicos**: Pendiente comunidad
- **Contributors**: [Ver lista completa](https://github.com/odin-distributed-systems/guide/graphs/contributors)

### 📚 Referencias Académicas

Basado en trabajos fundamentales de:
- **Leslie Lamport** (Paxos, Relojes Lógicos)
- **Barbara Liskov** (Viewstamped Replication)  
- **Eric Brewer** (CAP Theorem)
- **Martin Kleppmann** (Designing Data-Intensive Applications)
- **Nancy Lynch** (Distributed Algorithms)

## 📄 Licencia

Este proyecto está licenciado bajo [MIT License](LICENSE) - ver archivo para detalles.

### 🎯 Permisos
- ✅ Uso comercial
- ✅ Modificación  
- ✅ Distribución
- ✅ Uso privado

### 📋 Limitaciones
- ❌ Responsabilidad
- ❌ Garantía

## 🔗 Enlaces Útiles

### 📖 Documentación Técnica
- [MkDocs Material](https://squidfunk.github.io/mkdocs-material/)
- [OpenTelemetry Docs](https://opentelemetry.io/docs/)
- [TLA+ Documentation](https://lamport.azurewebsites.net/tla/tla.html)

### 🛠️ Herramientas Relacionadas
- [Jepsen](https://github.com/jepsen-io/jepsen) - Distributed systems testing
- [jcstress](https://github.com/openjdk/jcstress) - Java concurrency stress tests
- [Chaos Monkey](https://github.com/Netflix/chaosmonkey) - Chaos engineering

### 📚 Papers Fundamentales
- [Time, Clocks, and the Ordering of Events](https://lamport.azurewebsites.net/pubs/time-clocks.pdf)
- [The Part-Time Parliament](https://lamport.azurewebsites.net/pubs/lamport-paxos.pdf)
- [Harvest, Yield, and Scalable Tolerant Systems](https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.24.3690&rep=rep1&type=pdf)

---

<div align="center">

**[📖 Comenzar Lectura](docs/index.md)** | **[🚀 Ejemplos de Código](docs/recursos/ejemplos.md)** | **[💬 Discusiones](https://github.com/odin-distributed-systems/guide/discussions)**

---

*"La simplicidad es la máxima sofisticación en sistemas distribuidos"*

⭐ **Si este proyecto te resulta útil, dale una star!** ⭐

</div>
