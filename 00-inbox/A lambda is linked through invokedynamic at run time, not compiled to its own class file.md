---
tags: [java, java-core, lambda, jvm, interview]
status: draft
author: claude
source: "https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/invoke/LambdaMetafactory.html"
created: 2026-09-30
score: 0.873
review: "ready"
score_reasons: []
judged_by: "jev-1.13.0"
judge_rounds: 1
---
# A lambda is linked through invokedynamic at run time, not compiled to its own class file

## Core idea
For an anonymous class, `javac` writes a separate class file whose binary name is the enclosing
class name, `$` and a number, such as `Outer$1.class`. A lambda expression gets no class file of
its own: `javac` compiles the body into a private synthetic method of the enclosing class, such
as `lambda$run$0`, and emits an `invokedynamic` instruction where the lambda is evaluated. The
bootstrap method of that call site is `LambdaMetafactory.metafactory`, which at run time links it
to an object that implements the functional interface by delegating to the private method.

## Why choose / why not
- Choose a lambda when: the target type is a functional interface and the body is short; the
  JAR gets no extra class file per lambda.
- Choose an anonymous class when: the target is an abstract class or an interface with more than
  one abstract method, which a lambda cannot implement, or the object needs fields of its own.
- Prefer a method reference or a named method when: the body grows past a few lines; stack
  traces show only the synthetic name, such as `lambda$run$0`, which says nothing about intent.

## Interview angle
- Probed as "is a lambda just syntax sugar for an anonymous inner class?"; the expected answer is
  no, because the bytecode is different.
- Common wrong answer: "each lambda compiles to an inner class file such as `Outer$1.class`."
- Strong answer: private synthetic method, `invokedynamic`, `LambdaMetafactory` as bootstrap, and
  the reason for the design: the class file holds a recipe, so the JDK can change the translation
  strategy without recompiling user code.

## Related
- [[Java core MOC]]: this is the "what does the JVM do with this code?" question for
  lambdas in the Java core section of the map.
- [[JVM MOC]]: `invokedynamic` linkage and run-time class generation are JVM mechanics, so the
  note also belongs on the JVM topic map.
