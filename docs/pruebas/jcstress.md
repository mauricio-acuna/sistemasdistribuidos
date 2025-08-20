---
title: "Testing de Concurrencia con jcstress"
description: "Herramientas y técnicas para validar correctness en código concurrente"
nav_order: 1
---

# Testing de Concurrencia con jcstress

## 🎯 Objetivo

**jcstress** (Java Concurrency Stress tests) es el framework de referencia para encontrar bugs de concurrencia en código Java mediante testing exhaustivo de interleavings.

---

## 🚀 Setup Básico

### Maven Configuration

`xml
<dependency>
    <groupId>org.openjdk.jcstress</groupId>
    <artifactId>jcstress-core</artifactId>
    <version>0.16</version>
    <scope>test</scope>
</dependency>
`

### Test Simple

`java
@JCStressTest
@Outcome(id = "1, 1", expect = Expect.ACCEPTABLE, desc = "Both actors increment")
@Outcome(id = "1, 2", expect = Expect.ACCEPTABLE, desc = "Second actor sees first")  
@Outcome(id = "2, 1", expect = Expect.ACCEPTABLE, desc = "First actor sees second")
@Outcome(id = "2, 2", expect = Expect.ACCEPTABLE, desc = "Both see both increments")
@State
public class BasicCounterTest {
    int x = 0;
    
    @Actor
    public void actor1(II_Result r) {
        r.r1 = ++x;
    }
    
    @Actor  
    public void actor2(II_Result r) {
        r.r2 = ++x;
    }
}
`

---

## 📚 Más Recursos

La guía completa incluiría:

- Property-based testing
- Chaos engineering
- TLA+ specifications
- Performance testing estrategias

---

## 🔗 Referencias

- [jcstress Samples](https://github.com/openjdk/jcstress/tree/master/jcstress-samples/src/main/java/org/openjdk/jcstress/samples)
- [Concurrency Testing Guide](https://github.com/code-review-checklists/java-concurrency)
